---
name: negative-control-metabolite-selection
description: Use when applying hRUV/RUV-III normalisation to large-scale metabolomics
  data and you must decide which metabolites serve as negative controls — e.g. selecting
  all metabolites vs. ranking non-differentially expressed metabolites by p-values
  per batch for a response variable such as hypertension.
when_to_use_negative:
- When the study is small-scale or has no signal drift/batch structure — the assumption
  that pooled QC samples are identical over extended periods may hold, and negative-control
  sensitivity analysis adds no value.
- When the response variable and batch boundaries differ from a large-scale, multi-day
  design (e.g. a single short-run LC-MS/MS batch) — the p-value-per-batch ranking
  of non-differentially expressed metabolites cannot be estimated meaningfully.
- When metabolite data contain missing values that cannot be imputed — hRUV expects
  data without missing values, which is often not possible in large-scale metabolomics;
  the imputation workaround must be resolved first.
edam_operation: http://edamontology.org/operation_3653
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: hRUV
  role: Performs hierarchical removal of unwanted variation via RUV-III using the
    chosen negative-control metabolite set; the sensitivity analysis compares its
    normalised outputs under all-metabolites vs. data-driven negative controls.
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Environment for loading the normalised matrices, generating the two negative-control
    configurations, and scoring/tabulating the differences.
- name: hRUV Shiny application
  role: Interactive front-end for running hRUV with the selected negative-control
    metabolites when a scripted R package workflow is not required.
  repo: http://shiny.maths.usyd.edu.au/hRUV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:89bb333ba47b7a1f6079d80c12007e6bb39318f5f3f71d6eeae62e7be4215cb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - R
  - hRUV
  - hRUV Shiny application
  edam_operation: http://edamontology.org/operation_3653
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# negative-control-metabolite-selection

## Summary

Selects metabolite negative controls for RUV-III-based hRUV batch correction, comparing a data-driven iterative selection of stable (non-differentially expressed, p-value-ranked) metabolites against the default of using all metabolites, so the analyst can decide whether control-choice materially changes the normalized BioHEART-CT plasma metabolomics output.

## When to use

Use when applying hRUV/RUV-III normalisation to large-scale metabolomics data and you must decide which metabolites serve as negative controls — e.g. selecting all metabolites vs. ranking non-differentially expressed metabolites by p-values per batch for a response variable such as hypertension.

## When NOT to use

- When the study is small-scale or has no signal drift/batch structure — the assumption that pooled QC samples are identical over extended periods may hold, and negative-control sensitivity analysis adds no value.
- When the response variable and batch boundaries differ from a large-scale, multi-day design (e.g. a single short-run LC-MS/MS batch) — the p-value-per-batch ranking of non-differentially expressed metabolites cannot be estimated meaningfully.
- When metabolite data contain missing values that cannot be imputed — hRUV expects data without missing values, which is often not possible in large-scale metabolomics; the imputation workaround must be resolved first.

## Inputs

- hRUV-normalised (or pre-normalisation) metabolomics data matrices for the BioHEART-CT discovery cohort (>1,000 human plasma samples run over 44 days)
- Per-batch p-value ranks of non-differentially expressed metabolites for the response variable of interest (e.g. hypertension)
- hRUV configuration parameters (batch structure, replicate layout, hierarchical merging tree — balanced tree or concatenating design)

## Outputs

- Comparison table (e.g. negative_control_hierarchy_sensitivity.csv) scoring each negative-control configuration on removal of unwanted variation and inter-batch replicate utility
- hRUV normalised output matrices for each negative-control setting
- A significance assessment of the difference between data-driven and all-metabolite control sets

## How to apply

Load the hRUV-normalised metabolomics data matrices for the BioHEART-CT discovery cohort in R and generate hRUV outputs under two negative-control settings: (1) all metabolites, and (2) a data-driven set obtained by ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable. Score each configuration on the criteria the paper uses — removal of unwanted variation and utility of inter-batch sample replicates for batch correction, as tabulated in Supplementary Fig. 7 — then diff the resulting normalised matrices and quantify whether differences between the two control choices are significant. Rationale: metabolite signals depend on individual chemical properties, so appropriate negative-control selection is a challenge; the article's sensitivity analysis on BioHEART-CT plasma data found no significant differences between the two approaches, which justifies falling back to all metabolites when the data-driven selection is impractical.

## Related tools

- **hRUV** (Performs hierarchical removal of unwanted variation via RUV-III using the chosen negative-control metabolite set; the sensitivity analysis compares its normalised outputs under all-metabolites vs. data-driven negative controls.) — https://github.com/SydneyBioX/hRUV
- **R** (Environment for loading the normalised matrices, generating the two negative-control configurations, and scoring/tabulating the differences.)
- **hRUV Shiny application** (Interactive front-end for running hRUV with the selected negative-control metabolites when a scripted R package workflow is not required.) — http://shiny.maths.usyd.edu.au/hRUV

## Examples

```
Rscript -e 'library(hRUV); hRUV::runHruv(data = metabolomics_matrix, negative_controls = "all", response = "hypertension")'
```

## Evaluation signals

- The comparison table contains a row for each negative-control configuration (all metabolites and data-driven) with scores on removal of unwanted variation and utility of inter-batch sample replicates.
- The hRUV normalised matrices produced under the two control settings differ little; the paper found 'no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates'.
- The data-driven set is restricted to metabolites not flagged as differentially expressed when ranked by p-values per batch for the hypertension response variable.
- Output CSV (negative_control_hierarchy_sensitivity.csv) is populated and the significance test of the difference between configurations is reported.

## Limitations

- Selection of appropriate negative-control metabolites is challenging because metabolite signals depend on their individual chemical properties, so the data-driven ranking may not generalise to other responses or platforms.
- hRUV expects data without missing values, which is often not possible in large-scale metabolomics, so an imputation workaround is required before negative-control selection.
- Replicate selection can by chance draw samples clustered from only one half of the previous plate, limiting ability to capture unwanted variability across the whole plate — a separate design issue that interacts with control-choice sensitivity.

## Evidence

- [discussion] no significant differences between the two approaches in removal of unwanted variation: "no significant differences between the two approaches in removal of unwanted variation and utility of inter-batch sample replicates were more effective for batch correction"
- [discussion] the signals of the metabolites are dependent on their individual chemical properties: "the signals of the metabolites are dependent on their individual chemical properties4,11 and thus the selection of appropriate negative controls to correct for batch effect is a challenge"
- [discussion] While the proposed hRUV algorithm expects data without missing values: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [discussion] ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable: "ranking non-differentially expressed metabolites by p-values per batch for the hypertension response variable"
- [readme] The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III.: "The tool utilises 2 types of replicates: intra-batch and inter-batch replicates to estimate the unwatned variation within and between batches with RUV-III."
