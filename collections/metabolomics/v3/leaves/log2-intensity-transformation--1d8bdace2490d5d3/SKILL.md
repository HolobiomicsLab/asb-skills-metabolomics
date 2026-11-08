---
name: log2-intensity-transformation
description: Use when a raw metabolite abundance matrix (e.g. AUC peak areas integrated
  in MultiQuant from LC-MS/MS runs) must be placed on an additive scale before within-batch
  signal-drift correction or RUV-III batch adjustment in the hRUV pipeline.
when_to_use_negative:
- The input matrix is already on a log2 (or otherwise additive/log-scale) basis —
  re-transforming would distort the intensities.
- The matrix still contains zeros, negative values, or unresolved missing entries
  that have not been imputed; log2 is undefined there and hRUV's authors note that
  'hRUV algorithm expects data without missing values'.
- The goal is analysis of raw vendor peak tables rather than normalization downstream
  — the transformation alone removes no drift; it only prepares data for the loess/rlm
  smoother and RUV-III steps.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: hRUV
  role: R package whose pipeline consumes the log2-transformed abundance matrix for
    per-batch signal-drift correction with robust linear or non-linear smoothers and
    subsequent RUV-III batch adjustment
  repo: https://github.com/SydneyBioX/hRUV
- name: R
  role: Implementation language in which the element-wise log2 transformation of the
    metabolite abundance matrix is performed (R >= 4.0.3 per README)
- name: MultiQuant (SCIEX)
  role: Upstream tool producing the raw abundance peak areas (AUC for calibrated peaks)
    that are log2-transformed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2020.12.21.423723
    title: 'hRUV: Hierarchical approach to removal of unwanted variation for large-scale
      metabolomics data'
  source_content_hash: sha256:1bcc9cd988f8d10b7b86049d652178354a64c81f166217d8705e0eb4d7715d1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/hRUV
  tools:
  - MultiQuant (SCIEX)
  - R
  - hRUV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# log2-intensity-transformation

## Summary

Log2-transformation of raw metabolite abundance intensities as the preprocessing step of the hRUV workflow, converting multiplicative signal drift and ion-suppression effects into an additive scale so that per-batch robust smoothers (loess/rlm) against run order can be fitted and subtracted.

## When to use

Use when a raw metabolite abundance matrix (e.g. AUC peak areas integrated in MultiQuant from LC-MS/MS runs) must be placed on an additive scale before within-batch signal-drift correction or RUV-III batch adjustment in the hRUV pipeline.

## When NOT to use

- The input matrix is already on a log2 (or otherwise additive/log-scale) basis — re-transforming would distort the intensities.
- The matrix still contains zeros, negative values, or unresolved missing entries that have not been imputed; log2 is undefined there and hRUV's authors note that 'hRUV algorithm expects data without missing values'.
- The goal is analysis of raw vendor peak tables rather than normalization downstream — the transformation alone removes no drift; it only prepares data for the loess/rlm smoother and RUV-III steps.

## Inputs

- Raw metabolite abundance matrix (metabolites x samples; area-under-the-curve peak areas from MultiQuant/SCIEX Qtrap5500 LC-MS/MS)
- Per-well annotation: batch/plate identifier and run order for the BioHEART-CT plasma samples (e.g. 96-well plate layout with pooled QC every 10 runs)

## Outputs

- Log2-transformed metabolite-by-sample intensity matrix with batch and run-order annotations preserved, ready for within-batch smoother fitting

## How to apply

Start from the raw metabolite-by-sample abundance matrix produced by integrating calibrated LC-MS/MS peak areas in MultiQuant (SCIEX), after metabolites absent from at least 50% of samples have been filtered out and outlying measurements flagged from the three consecutive pooled QC samples at the start of each 96-well plate. Apply log2() element-wise to every abundance value, producing an adjusted log2 intensity matrix; because log2 is undefined at zero and negative values, missing or non-positive entries must be handled (e.g. imputed) beforehand, consistent with hRUV's expectation of complete data. Retain and carry forward the batch label and run-order annotation for every well, since the log2 matrix is the direct input to the next step, where a robust smoother — loess or a robust linear model (rlm) via MASS — is fitted per metabolite of log2 signal against run order within each batch and subtracted to remove drift. The rationale is that run-order effects and matrix/ionisation suppression act multiplicatively on raw intensities, so the log scale makes them additive and removable by subtraction of the fitted smoother.

## Related tools

- **hRUV** (R package whose pipeline consumes the log2-transformed abundance matrix for per-batch signal-drift correction with robust linear or non-linear smoothers and subsequent RUV-III batch adjustment) — https://github.com/SydneyBioX/hRUV
- **R** (Implementation language in which the element-wise log2 transformation of the metabolite abundance matrix is performed (R >= 4.0.3 per README))
- **MultiQuant (SCIEX)** (Upstream tool producing the raw abundance peak areas (AUC for calibrated peaks) that are log2-transformed)

## Examples

```
log2_abund <- log2(as.matrix(raw_metabolite_abundance))  # element-wise log2 of the metabolite-by-sample AUC matrix before hRUV drift correction
```

## Evaluation signals

- No NaN/-Inf values are produced: every raw abundance entry in the filtered matrix is strictly positive before log2.
- The transformed matrix has identical dimensions (metabolites x samples) and identical row/column ordering to the raw matrix, with batch and run-order columns unchanged.
- Per-batch boxplots of log2 intensities across run order show the multiplicative run-order drift now expressed as an approximately linear/additive trend, which the subsequent loess/rlm smoother can capture.
- Downstream hRUV replicate-based diagnostics (low variability among inter-batch sample replicates, retention of biological signal) remain consistent with the paper's reported hRUV performance.
- Values agree with the log2-transformed matrix used in the SydneyBioX/BioHEART_metabolomics analysis scripts when re-run on the same BioHEART-CT inputs.

## Limitations

- The transformation does not itself correct signal drift; it must be followed by the per-batch robust smoother (loess or rlm) fitted against run order.
- log2 is undefined for zeros/negatives, and the paper notes hRUV 'expects data without missing values, which is often not possible in large-scale metabolomics data', requiring an imputation workaround before or after transformation.
- Residual drift between specific batches (e.g. batches 6-7 and 13-14, where replicate clustering limited plate-wide coverage) can persist even after log2-based drift correction and RUV-III adjustment.

## Evidence

- [intro] applying signal drift correction with robust linear or non-linear smoothers: "We propose a novel hierarchical approach to removing unwanted variation by harnessing information from sample replicates embedded in the sequence of experimental runs/batches and applying signal"
- [discussion] metabolites that were not present in at least 50% of the samples were filtered out: "metabolites that were not present in at least 50% of the samples were filtered out"
- [discussion] While the proposed hRUV algorithm expects data without missing values: "While the proposed hRUV algorithm expects data without missing values, this is often not possible in large-scale metabolomics data"
- [readme] hRUV ... applying signal drift correction with robust linear or non-linear smoothers: "Our novel tool is a novel hierarchical approach to removing unwanted variation by harnessing information from sample replicates embedded in the seequence of experimental runs/batches and applying"
