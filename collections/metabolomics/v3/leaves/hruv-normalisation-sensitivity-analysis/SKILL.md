---
name: hruv-normalisation-sensitivity-analysis
description: Use when hRUV-normalised metabolomics data must be robustness-checked
  against alternative negative-control metabolite selections or alternative hierarchical
  batch-merging (tree) designs.
when_to_use_negative:
- The data are single-batch or lack embedded inter-batch sample replicates — hRUV's
  hierarchical merging and the negative-control/tree sensitivity axis have nothing
  to vary
- The goal is benchmarking hRUV against the 13 external normalisation methods (MetaboAnalyst,
  NormalyzerDE, etc.) rather than testing internal hRUV configuration choices
- The input matrix still contains missing values — hRUV expects complete data, so
  an imputation step (e.g. DMwR2/impute packages from the README) must precede any
  sensitivity comparison
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: hRUV
  role: Run normalisation under each negative-control and hierarchical-merging configuration;
    available as R package and Shiny app
  repo: github.com/SydneyBioX/hRUV
- name: R
  role: 'Implement the comparison workflow: generate configurations, score criteria,
    and produce the sensitivity CSV'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:f338b1b94c47bc48472133dccaafa18e6517d7dcc8e54e91fc4caf660b406258
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hruv-normalisation-sensitivity-analysis

## Summary

A sensitivity-analysis skill for hRUV normalisation: stress-test whether normalised outputs change meaningfully when negative-control metabolites are chosen differently (all metabolites vs data-driven stable metabolites) or when the hierarchical batch-merging structure changes (balanced tree vs concatenation).

## When to use

Use when hRUV-normalised metabolomics data must be robustness-checked against alternative negative-control metabolite selections or alternative hierarchical batch-merging (tree) designs.

## When NOT to use

- The data are single-batch or lack embedded inter-batch sample replicates — hRUV's hierarchical merging and the negative-control/tree sensitivity axis have nothing to vary
- The goal is benchmarking hRUV against the 13 external normalisation methods (MetaboAnalyst, NormalyzerDE, etc.) rather than testing internal hRUV configuration choices
- The input matrix still contains missing values — hRUV expects complete data, so an imputation step (e.g. DMwR2/impute packages from the README) must precede any sensitivity comparison

## Inputs

- hRUV-normalised metabolomics abundance matrix for a multi-batch cohort (e.g. BioHEART-CT discovery cohort plasma samples)
- Negative-control definitions: all metabolites, or non-differentially expressed metabolites ranked by p-values per batch for the response variable
- Batch/plate replicate annotations (intra-batch and inter-batch replicates used by RUV-III)
- Hierarchical merging design specification (balanced tree vs concatenating design, Figure 1d)

## Outputs

- Comparison table negative_control_hierarchy_sensitivity.csv tabulating differences across negative-control and tree-structure configurations
- Per-configuration scores on the Supplementary Fig 7 criteria (removal of unwanted variation; utility of inter-batch sample replicates)

## How to apply

Load the hRUV-normalised metabolomics data matrices for the cohort of interest (in the article, the BioHEART-CT discovery cohort of over 1,000 human plasma samples run over 44 days). Run hRUV — which estimates unwanted variation with RUV-III and progressively merges adjustments of neighbouring batches — under two negative-control settings: (a) all metabolites as negative controls, and (b) data-driven negative controls defined by ranking non-differentially expressed metabolites by p-values per batch for the response variable (hypertension in the article). Separately, generate hRUV outputs under two hierarchical merging structures: the balanced tree and the concatenating design illustrated in Figure 1d. Score each configuration on the Supplementary Fig 7 criteria — removal of unwanted variation and utility of inter-batch sample replicates for batch correction — then tabulate the differences across all configurations, assess whether they are significant, and save the comparison table as negative_control_hierarchy_sensitivity.csv. The article's reference result: no significant differences between the two negative-control approaches, with inter-batch sample replicates being more effective for batch correction.

## Related tools

- **hRUV** (Run normalisation under each negative-control and hierarchical-merging configuration; available as R package and Shiny app) — github.com/SydneyBioX/hRUV
- **R** (Implement the comparison workflow: generate configurations, score criteria, and produce the sensitivity CSV)

## Examples

```
if (!("devtools" %in% rownames(installed.packages()))) install.packages("devtools"); library(devtools); devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- The comparison table contains one entry per configuration (2 negative-control settings × 2 tree structures) with scores on both Supplementary Fig 7 criteria
- Reproduces the article's expected outcome: little to no significant difference between data-driven and all-metabolite negative controls in removal of unwanted variation
- Data-driven negative controls are verifiably non-differentially expressed metabolites ranked by p-values per batch for the hypertension response variable
- Each configuration uses the same replicate design (pooled QCs plus intra-/inter-batch replicates), so only the negative-control set or tree structure differs
- Output CSV is saved with the expected filename negative_control_hierarchy_sensitivity.csv

## Limitations

- Selection of appropriate negative-control metabolites is inherently challenging because metabolite signals depend on their individual chemical properties
- hRUV expects data without missing values, which is often not possible in large-scale metabolomics and requires an imputation workaround
- The Supplementary Fig 7 criteria and the Figure 1d tree/concatenation schematics are referenced but not included in the article text, so scoring must recover them from supplementary material or code
- Interpretation depends on plate-wide replicate coverage: unintended clustering of batch replicates (between batches 6–7 and 13–14) limited capture of unwanted variability across the whole plate

## Evidence

- [discussion] no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
- [discussion] Non-differentially expressed metabolites ranked by p-values per batch for adaptive negative-control selection: "ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable"
- [abstract] hRUV removes unwanted variation hierarchically by merging neighbouring batch adjustments: "We integrate these replicates in a novel workflow to remove unwanted variation in a hierarchical structure (hRUV) by progressively merging the adjustments in neighbouring batches"
- [readme] hRUV is an R package for hierarchical normalisation of multi-batch metabolomics data using sample replicates: "is a package for normalisation of multiple batches of metabolomics data in a hierarchical strategy with use of samples replicates"
