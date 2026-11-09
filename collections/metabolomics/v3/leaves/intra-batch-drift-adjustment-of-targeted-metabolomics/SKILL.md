---
name: intra-batch-drift-adjustment-of-targeted-metabolomics
description: Use when a targeted metabolomics run sequence shows run-order-dependent
  signal drift within a batch/plate (visible in pooled QC or replicate traces over
  run order) and drift must be removed before between-batch correction.
when_to_use_negative:
- The analysis goal is between-batch variation removal — that is handled by RUV-III
  with embedded sample replicates in hRUV, not by the intra-batch smoother
- The matrix still contains substantial missing values and no imputation strategy
  (e.g. the DMwR2/impute packages) has been decided — hRUV expects data without missing
  values
- Drift is attributable to pooled-QC degradation over months/years between batches
  rather than run-order effects within a single plate; the intra-batch smoother does
  not address that assumption failure
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: hRUV
  role: R package implementing the hierarchical workflow whose first stage is signal
    drift correction with robust linear or non-linear smoothers
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Implementation environment; fitting of per-metabolite loess/rlm smoothers
    of log2 signal vs run order within each batch
- name: MASS
  role: Supplies rlm, the robust linear smoother option for drift correction (listed
    as a CRAN dependency of hRUV)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:78f42fd6e0061b5f27bd0299f0a899e89721a5903f83011432481ba4450b4786
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - MASS
  - R
  - hRUV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# intra-batch-drift-adjustment-of-targeted-metabolomics

## Summary

Corrects within-batch signal drift in targeted metabolomics by fitting, per metabolite, a robust smoother (loess or robust linear model rlm) of log2 signal against run order and subtracting the fitted trend, as done in the hRUV workflow on BioHEART-CT plasma data.

## When to use

Use when a targeted metabolomics run sequence shows run-order-dependent signal drift within a batch/plate (visible in pooled QC or replicate traces over run order) and drift must be removed before between-batch correction.

## When NOT to use

- The analysis goal is between-batch variation removal — that is handled by RUV-III with embedded sample replicates in hRUV, not by the intra-batch smoother
- The matrix still contains substantial missing values and no imputation strategy (e.g. the DMwR2/impute packages) has been decided — hRUV expects data without missing values
- Drift is attributable to pooled-QC degradation over months/years between batches rather than run-order effects within a single plate; the intra-batch smoother does not address that assumption failure

## Inputs

- Raw metabolite abundance matrix (metabolites x samples) from targeted LC-MS/MS (e.g. Agilent 1260 HPLC + Sciex Qtrap5500, peak areas integrated in MultiQuant)
- Per-well annotation of batch (96-well plate) and run order
- Run-order index within each batch

## Outputs

- Drift-adjusted log2 metabolite-by-sample intensity matrix
- Preserved batch and run-order annotations aligned to the adjusted matrix
- (Optionally) fitted smoother values per metabolite for QC diagnostics

## How to apply

Load the raw metabolite abundance matrix with run-order and batch annotation for each well and log2-transform the abundances. For each batch separately, fit a smoother of log2 signal against run order for every metabolite — either a robust linear model (MASS::rlm) or a non-linear local regression (loess); the hRUV paper explores both robust linear and non-linear smoothers for removing run-order effects. Because each metabolite drifts differently, the smoother must be fitted per metabolite rather than on a pooled profile. Subtract the fitted values from the observed log2 signals to yield drift-adjusted log2 intensities, preserving batch and run-order annotations for the downstream hierarchical RUV-III between-batch step. Prior filtering matters: metabolites absent from at least 50% of samples were removed in the BioHEART-CT workflow, and hRUV ultimately expects a complete (imputed) matrix.

## Related tools

- **hRUV** (R package implementing the hierarchical workflow whose first stage is signal drift correction with robust linear or non-linear smoothers) — https://github.com/SydneyBioX/hRUV
- **R** (Implementation environment; fitting of per-metabolite loess/rlm smoothers of log2 signal vs run order within each batch)
- **MASS** (Supplies rlm, the robust linear smoother option for drift correction (listed as a CRAN dependency of hRUV))

## Examples

```
fit <- loess(log2_abundance ~ run_order, data = batch_df, family = "symmetric"); adj <- batch_df$log2_abundance - fitted(fit)
```

## Evaluation signals

- After adjustment, per-metabolite log2 signal shows no residual monotonic trend against run order within each batch (plot adjusted intensity vs run order; the run-order pattern visible before fitting should disappear)
- Adjusted values equal observed log2 signal minus fitted smoother values exactly for each metabolite-batch combination
- Embedded replicates (pooled QCs, short within-row replicates, 5 batch replicates per plate) show reduced within-batch variability after drift adjustment
- Biological signal is retained — compare replicate variability and known biological contrasts before/after as in the hRUV assessment criteria
- Output matrix schema: same metabolite x sample dimensions as input, with batch and run-order annotation columns preserved

## Limitations

- hRUV (and hence the pipeline this step feeds) expects data without missing values, which is often not possible in large-scale metabolomics, requiring an imputation workaround
- Choice of smoother (rlm vs loess) is a decision point; the paper explored both but gives no universal rule for selection
- This step only addresses within-batch run-order drift; residual between-batch shifts (e.g. the slight shift observed between batches 6/7 and 13/14) must be handled by the subsequent RUV-III step

## Evidence

- [readme] hRUV applies robust smoothers per batch: "a novel hierarchical approach to removing unwanted variation by harnessing information from sample replicates embedded in the seequence of experimental runs/batches and applying signal drift"
- [intro] Gap motivating the smoother step: "most of them adapt common normalization methods developed for other omics platforms and do not account for signal drift across extended time or inter-plate variations"
- [discussion] Pre-filter threshold before drift correction: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] Missing-value limitation: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [intro] Evaluation criterion for the normalization: "we utilise this to assess the normalization on a number of criteria including retention of biological signal, low variability among replication, and reproducibility of results in comparison to other"
