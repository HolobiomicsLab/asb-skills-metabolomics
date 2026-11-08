---
name: hierarchical-batch-merging-structure-comparison
description: Use when hRUV-normalising a large-scale metabolomics cohort (e.g. BioHEART-CT
  plasma batches) and you must justify the choice of negative-control metabolites
  or the hierarchical merging design rather than accepting defaults.
when_to_use_negative:
- Input is raw unprocessed metabolomics signal — signal drift correction, replicate
  embedding (plateDesign function), and the ≥50% sample-presence metabolite filter
  must be applied via hRUV before comparing merging structures
- Data contains missing values and imputation is not planned — the authors note hRUV
  expects data without missing values, which requires an imputation workaround (DMwR2/impute
  packages)
- Single-batch or small-scale data with no inter-batch replicates — the comparison
  of merging structures and batch-correction utility is meaningless without replicate-embedded
  batches
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: hRUV
  role: Generates normalised outputs under each negative-control and hierarchical
    merging configuration for comparison
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Analysis environment for loading matrices, running configurations, scoring,
    and writing the comparison CSV; randomization performed with the R sample function
- name: hRUV (R package, plateDesign function)
  role: Provides the replicate design underlying the inter-batch sample replicates
    scored in the comparison
  repo: https://github.com/SydneyBioX/hRUV
- name: hRUV Shiny application
  role: Web interface exposing the same hierarchical normalisation workflow at shiny.maths.usyd.edu.au/hRUV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:512f31c0778a320223a7770ec8a642f2cc332dba578adc8f38ae436bbe4fa374
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  - hRUV (R package, plateDesign function)
  - hRUV Shiny application
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hierarchical-batch-merging-structure-comparison

## Summary

Benchmarks hRUV normalisation outputs across alternative hierarchical batch-merging structures (balanced tree vs concatenating design) and negative-control settings (data-driven stable metabolites vs all metabolites), scoring each configuration on unwanted-variation removal and inter-batch replicate utility.

## When to use

Use when hRUV-normalising a large-scale metabolomics cohort (e.g. BioHEART-CT plasma batches) and you must justify the choice of negative-control metabolites or the hierarchical merging design rather than accepting defaults.

## When NOT to use

- Input is raw unprocessed metabolomics signal — signal drift correction, replicate embedding (plateDesign function), and the ≥50% sample-presence metabolite filter must be applied via hRUV before comparing merging structures
- Data contains missing values and imputation is not planned — the authors note hRUV expects data without missing values, which requires an imputation workaround (DMwR2/impute packages)
- Single-batch or small-scale data with no inter-batch replicates — the comparison of merging structures and batch-correction utility is meaningless without replicate-embedded batches

## Inputs

- hRUV-normalised metabolomics data matrices for the BioHEART-CT discovery cohort (over 1,000 human plasma samples run over 44 days)
- Batch/plate replicate layout: pooled QC samples (3 at plate start, then one every 10 runs), 5 batch replicates from the previous plate, and one short replicate per row
- Negative-control definitions: data-driven list (non-differentially expressed metabolites ranked by p-values per batch) and the all-metabolites alternative
- Hierarchical merging structure definitions: balanced tree and concatenating design (Figure 1d)

## Outputs

- negative_control_hierarchy_sensitivity.csv — comparison table of configuration scores
- Per-configuration scores on removal of unwanted variation and utility of inter-batch sample replicates (Supplementary Fig 7 criteria)
- Significance assessment of differences between negative-control and hierarchy configurations

## How to apply

In R, load the hRUV-normalised matrices for the BioHEART-CT discovery cohort and regenerate normalisation outputs under each configuration: (a) negative controls defined either as data-driven iterative selections — non-differentially expressed metabolites ranked by p-values per batch for the hypertension response variable — or as all metabolites; (b) hierarchical merging structures of the balanced tree or the concatenating design illustrated in Figure 1d. Score each configuration against the Supplementary Fig 7 criteria — removal of unwanted variation and utility of inter-batch sample replicates for batch correction. Tabulate differences across all configurations and assess significance; the reference result was that no significant differences were seen between the two negative-control approaches, and inter-batch sample replicates were more effective for batch correction. Save the comparison as negative_control_hierarchy_sensitivity.csv.

## Related tools

- **hRUV** (Generates normalised outputs under each negative-control and hierarchical merging configuration for comparison) — https://github.com/SydneyBioX/hRUV
- **R** (Analysis environment for loading matrices, running configurations, scoring, and writing the comparison CSV; randomization performed with the R sample function)
- **hRUV (R package, plateDesign function)** (Provides the replicate design underlying the inter-batch sample replicates scored in the comparison) — https://github.com/SydneyBioX/hRUV
- **hRUV Shiny application** (Web interface exposing the same hierarchical normalisation workflow at shiny.maths.usyd.edu.au/hRUV)

## Examples

```
devtools::install_github("SydneyBioX/hRUV", build_vignettes=TRUE, dependencies = TRUE)
```

## Evaluation signals

- Reference outcome: little to no significant difference between data-driven negative controls and all-metabolites negative controls in removal of unwanted variation
- Inter-batch sample replicates should show greater effectiveness for batch correction than the alternative configurations on the Supplementary Fig 7 criteria
- Comparison table negative_control_hierarchy_sensitivity.csv contains one scored row per configuration (2 negative-control settings × 2 hierarchy designs)
- Biological signal (e.g. hypertension response) should be retained while unwanted between-batch variation is reduced in the preferred configuration
- Input matrices must be complete (no missing values) after the ≥50%-presence metabolite filter and QC-outlier removal

## Limitations

- Supplementary Fig 7 (negative-control comparison criteria) and Figure 1d (balanced-tree vs concatenation schematics) are referenced but not included in the main text, complicating exact replication
- Selection of appropriate negative-control metabolites is inherently challenging because metabolite signals depend on their individual chemical properties
- Replicate selection can by chance cluster samples from only one half of the previous plate (observed between batches 6–7 and 13–14), limiting capture of plate-wide unwanted variability and leaving slight residual signal shifts for selected metabolites
- hRUV expects data without missing values, which is often not possible in large-scale metabolomics and requires imputation

## Evidence

- [discussion] no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
- [discussion] ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable: "ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable"
- [intro] We propose a novel hierarchical approach to removing unwanted variation: "We propose a novel hierarchical approach to removing unwanted variation by harnessing information from sample replicates embedded in the sequence of experimental runs/batches and applying signal"
- [readme] tested for macOS Big Sur and Linux Debian 10 with R 4.0.3; install from GitHub with devtools: [evidence span withheld — anchor status: external_doc]
- [discussion] metabolites not present in at least 50% of samples were filtered out: "metabolites that were not present in at least 50% of the samples were filtered out"
