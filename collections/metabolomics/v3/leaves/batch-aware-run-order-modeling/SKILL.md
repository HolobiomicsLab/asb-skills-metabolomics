---
name: batch-aware-run-order-modeling
description: Use when targeted metabolomics signal shows visible trends against run
  order within a batch and each metabolite drifts differently, so a single per-batch
  correction is insufficient.
when_to_use_negative:
- When the primary variation to correct is between batches rather than within-batch
  signal drift — use the downstream hRUV RUV-III inter-batch correction instead
- When input data have not yet been filtered/imputed and are not complete (hRUV expects
  data without missing values), or when metabolite presence does not meet the ≥50%
  sample filter
- When run-order trends are flat (no signal drift observed), fitting a smoother would
  inject noise rather than correct drift
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: hRUV
  role: R package providing the intra-batch signal-drift correction workflow (robust
    linear/loess smoothers) and downstream inter-batch RUV-III merging
  repo: github.com/SydneyBioX/hRUV
- name: R
  role: Implementation language; the smoother models (rlm, loess) are fitted in R
    within the hRUV package
  repo: https://github.com/SydneyBioX/hRUV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:eecd7055feac6287c90aecf96eb109c16b8bdd62754cfdfbcbd148da2449c77b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# batch-aware-run-order-modeling

## Summary

Fits a robust smoother (robust linear regression or loess) of log2-transformed metabolite signal against run order within each batch, then subtracts the fitted trend to remove intra-batch signal drift in large-scale targeted LC-MS/MS metabolomics data.

## When to use

Use when targeted metabolomics signal shows visible trends against run order within a batch and each metabolite drifts differently, so a single per-batch correction is insufficient.

## When NOT to use

- When the primary variation to correct is between batches rather than within-batch signal drift — use the downstream hRUV RUV-III inter-batch correction instead
- When input data have not yet been filtered/imputed and are not complete (hRUV expects data without missing values), or when metabolite presence does not meet the ≥50% sample filter
- When run-order trends are flat (no signal drift observed), fitting a smoother would inject noise rather than correct drift

## Inputs

- Raw metabolite-by-sample abundance matrix from targeted LC-MS/MS runs (e.g. Qtrap5500/Sciex, MultiQuant peak areas)
- Per-sample run order and batch/plate annotation

## Outputs

- Drift-adjusted log2 metabolite-by-sample matrix with batch and run-order annotations preserved

## How to apply

Load the raw metabolite-by-sample abundance matrix for the large-scale study (e.g. BioHEART-CT plasma samples run over 44 days) together with each well's run order and batch annotation. Log2-transform the metabolite abundance values, then, for each batch independently, fit a robust smoother — either a robust linear model (rlm via MASS) or a non-linear loess/local regression — of log2 signal against run order for each metabolite. Subtract the fitted smoother values from the observed log2 signals so that the within-batch drift is removed, producing a drift-adjusted log2 intensity matrix. Keep the batch and run-order annotations attached to the output, because they are required by the downstream hRUV inter-batch correction step (RUV-III merging of neighbouring batches).

## Related tools

- **hRUV** (R package providing the intra-batch signal-drift correction workflow (robust linear/loess smoothers) and downstream inter-batch RUV-III merging) — github.com/SydneyBioX/hRUV
- **R** (Implementation language; the smoother models (rlm, loess) are fitted in R within the hRUV package) — https://github.com/SydneyBioX/hRUV

## Examples

```
library(hRUV); drift_corrected <- intraNorm(log2_exprs = log2(ms_mat), run_order = meta$run_order, batch = meta$batch, method = "loess")
```

## Evaluation signals

- After correction, log2 metabolite signal should no longer show a visible trend against run order within each batch (inspect drift plots per metabolite/batch)
- Within-batch replicate variability should decrease relative to the uncorrected matrix while biological signal is retained (assess against replicate sets)
- The output matrix must be complete (no missing values) since downstream hRUV/RUV-III expects complete data
- Batch and run-order annotations must be preserved and aligned row-for-row with the corrected intensity matrix
- Compare smoother choices (rlm vs loess) by residual run-order trend; both should reduce drift without distorting biological contrasts

## Limitations

- hRUV expects data without missing values, so an imputation step (e.g. DMwR2/impute in the hRUV package) is needed before smoother fitting
- Only intra-batch drift is corrected here; residual between-batch shifts (e.g. between batches 6 and 7, 13 and 14) require the hierarchical RUV-III step
- The smoother cannot capture unwanted variability across the whole plate when replicate selection is unbalanced
- Outlying measurements should be removed based on the three consecutive QC samples at the start of each plate before fitting; uncorrected outliers will bias the smoother

## Evidence

- [discussion] Within each batch, signal drift is corrected using a robust smoother that captures the trends visible by run order, with both robust linear (rlm) and non-linear (loess/local regression) smoothers explored for removing run order effects.: [evidence span withheld — anchor status: not_found]
- [full_text] Workflow from source card task_003 (log2-transform, smoother fit, subtract, output).: [evidence span withheld — anchor status: not_found]
- [readme] Smoothers are applied within a hierarchical hRUV workflow embedded in the R package.: "a novel hierarchical approach to removing unwanted variation by harnessing information from sample replicates embedded in the seequence of experimental runs/batches and applying signal drift"
- [discussion] Missing values must be handled before correction; the package relies on impute/DMwR2.: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] Data completeness is a prerequisite for downstream batch merging.: "dplyr
  tidyr
  rlang
  ggplot2
  plotly
  S4Vectors
  tibble
  DMwR2
  MASS"
