---
name: robust-smoother-fitting-loess-rlm
description: Use when a metabolomics acquisition shows run-order-dependent signal
  drift within a batch, and a metabolite-by-sample log2 intensity matrix with run-order
  annotations is available to fit a drift correction smoother.
when_to_use_negative:
- Data has no run-order/batch annotation, so no smoother can be fit against run order
- Signal drift is absent or negligible within batches (e.g. very short acquisition
  with stable signal)
- Inter-batch batch effects are the dominant unwanted variation and hRUV-style within-batch
  drift correction is not needed before RUV-III
edam_operation: http://edamontology.org/operation_3217
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: hRUV
  role: R package that implements the robust linear/non-linear smoother signal-drift
    correction step within the hierarchical unwanted-variation workflow
  repo: github.com/SydneyBioX/hRUV
- name: R
  role: Runtime environment in which loess/rlm fitting and drift subtraction are performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:52687fc2a0d4cb34325464c0783e3dded6dc63b25f9a3cae4d5206c595b449f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - hRUV
  edam_operation: http://edamontology.org/operation_3217
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# robust-smoother-fitting-loess-rlm

## Summary

Fits a robust smoother — loess (local non-linear regression) or rlm (robust linear model) — of log2 metabolite signal against run order within each batch to capture and remove signal drift in metabolomics data, as implemented in the hRUV workflow.

## When to use

Use when a metabolomics acquisition shows run-order-dependent signal drift within a batch, and a metabolite-by-sample log2 intensity matrix with run-order annotations is available to fit a drift correction smoother.

## When NOT to use

- Data has no run-order/batch annotation, so no smoother can be fit against run order
- Signal drift is absent or negligible within batches (e.g. very short acquisition with stable signal)
- Inter-batch batch effects are the dominant unwanted variation and hRUV-style within-batch drift correction is not needed before RUV-III

## Inputs

- Raw metabolite-by-sample abundance matrix (e.g. BioHEART-CT plasma targeted LC-MS/MS peak areas from MultiQuant integration)
- Per-well run-order and batch annotation table
- Log2-transformed metabolite abundance matrix

## Outputs

- Drift-adjusted log2 metabolite-by-sample matrix
- Fitted smoother values per metabolite per batch

## How to apply

For each batch, fit a robust smoother of log2-transformed metabolite abundance against run order (well position in the batch sequence), exploring both loess for non-linear drift trends and rlm (MASS robust linear models) for linear trends; the smoother captures trends visible by run order that vary per metabolite. Subtract the fitted smoother values from the observed log2 signals to yield drift-adjusted log2 intensities, preserving batch and run-order annotations. In the hRUV context this within-batch drift correction is applied before inter-batch RUV-III normalization, so decide on loess vs rlm by whether drift appears linear or curved in run-order plots. The BioHEART-CT data used metabolites that passed a ≥50% detection filter before fitting.

## Related tools

- **hRUV** (R package that implements the robust linear/non-linear smoother signal-drift correction step within the hierarchical unwanted-variation workflow) — github.com/SydneyBioX/hRUV
- **R** (Runtime environment in which loess/rlm fitting and drift subtraction are performed)

## Examples

```
fit <- loess(log2_intensity ~ run_order, data = batch_df); adjusted <- log2_intensity - fitted(fit)
```

## Evaluation signals

- Drift trends visible by run order in pre-correction plots should be flattened in post-correction plots
- Per-metabolite residual vs run-order plots should show no systematic slope after smoother subtraction
- Within-batch variance among short/plate replicates should decrease after drift correction
- Adjusted matrix must retain the same sample × metabolite dimensions and batch/run-order annotations as the input
- Log2 scale must be preserved (inputs and outputs both log2 intensity)

## Limitations

- hRUV expects data without missing values, so imputation may be required before smoother fitting (e.g. via the impute Bioconductor package)
- Metabolites not present in at least 50% of samples were filtered out before correction, so fitting is only on detected features
- Residual slight shifts in signal can persist between adjacent batches for selected metabolites, so within-batch smoothers alone may not fully remove batch effects

## Evidence

- [results] Within each batch, signal drift is corrected using a robust smoother that captures the trends visible by run order: [evidence span withheld — anchor status: not_found]
- [intro] robust linear and non-linear smoothers are explored: "applying signal drift correction with robust linear or non-linear smoothers"
- [discussion] Metabolite detection filter before fitting: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] hRUV expects data without missing values: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] README describes the tool and its drift-correction role: "a novel hierarchical approach to removing unwanted variation by harnessing information from sample replicates embedded in the seequence of experimental runs/batches and applying signal drift"
