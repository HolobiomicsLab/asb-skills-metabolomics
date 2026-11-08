---
name: cubic-spline-regression-on-qc-injection-order
description: Use when untargeted LC-MS peak tables show systematic signal intensity
  drift over the injection sequence and QC samples are available across the run to
  model the drift as a smooth function of injection order.
when_to_use_negative:
- The dataset has no pooled QC samples injected across the run, so no drift model
  can be fit (features become Missing_QCs)
- The goal is batch effect correction between acquisition batches, which requires
  notame's normalize_batches / dobc / ruvs_qc strategies instead
- Data have not been preprocessed into a peak table; this skill operates on preprocessed
  LC-MS data, not raw instrument files
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: notame
  role: R package providing the correct_drift() function and MetaboSet object that
    implement the cubic-spline drift correction workflow
  repo: https://github.com/hanhineva-lab/notame
- name: R
  role: Language in which smooth.spline performs the drift modelling and in which
    notame runs
- name: smooth.spline
  role: Base-R cubic smoothing spline function used to model drift as a function of
    injection order on QC samples
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:5e4d86b86d43b64f383e1a4d623d72315cec3e17d2458a9b20ee4a8ece80d58f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  - smooth.spline
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cubic-spline-regression-on-qc-injection-order

## Summary

Drift correction for LC-MS feature intensities using the notame correct_drift() approach: fit an R smooth.spline cubic spline to QC samples' log-transformed abundances versus injection order, apply an additive correction in log space, and back-transform to the linear scale.

## When to use

Use when untargeted LC-MS peak tables show systematic signal intensity drift over the injection sequence and QC samples are available across the run to model the drift as a smooth function of injection order.

## When NOT to use

- The dataset has no pooled QC samples injected across the run, so no drift model can be fit (features become Missing_QCs)
- The goal is batch effect correction between acquisition batches, which requires notame's normalize_batches / dobc / ruvs_qc strategies instead
- Data have not been preprocessed into a peak table; this skill operates on preprocessed LC-MS data, not raw instrument files

## Inputs

- MetaboSet object (or peak table) with feature abundance matrix and sample metadata including injection order and QC labels
- QC sample annotations identifying quality-control injections spanning the run
- Preprocessed LC-MS peak table, e.g. MS-DIAL Excel export read via notame::read_from_excel

## Outputs

- Drift-corrected abundance matrix (linear scale), e.g. written to drift_corrected_exprs.csv
- Per-feature DC_note column with values Drift_corrected, Low-quality, Missing_QCs, or Negative_DC

## How to apply

First mark zero abundances as missing (notame::mark_nas) and log-transform the exprs matrix, since drift correction is performed on log-transformed data to satisfy the spline model assumptions. For each feature, fit smooth.spline to the QC samples' log-abundance versus injection order, selecting the smoothing parameter by leave-one-out cross validation within the recommended interval [0.5, 1.5]; require at least 4 QC samples, otherwise record the feature as Missing_QCs. Apply the additive correction x_corrected(i) = x_original(i) + mean_QC − f_drift(i) in log space for every sample, then reverse the log transformation to return to the linear scale. Record a DC_note per feature — Drift_corrected, Low-quality, Missing_QCs, or Negative_DC — depending on whether drift correction improved quality, QCs were insufficient, or the correction produced negative values.

## Related tools

- **notame** (R package providing the correct_drift() function and MetaboSet object that implement the cubic-spline drift correction workflow) — https://github.com/hanhineva-lab/notame
- **R** (Language in which smooth.spline performs the drift modelling and in which notame runs)
- **smooth.spline** (Base-R cubic smoothing spline function used to model drift as a function of injection order on QC samples)

## Examples

```
notame::read_from_excel(file = system.file("extdata", "sample_data_whole.xlsx", package = "notame"), sheet = 1, corner_row = 4, corner_column = "X", split_by = c("Column", "Ion mode")); correct_drift(mset)
```

## Evaluation signals

- drift_corrected_exprs.csv exists with one row per sample and one column per feature
- The DC_note column is present with expected values (Drift_corrected, Low-quality, Missing_QCs, Negative_DC)
- QC-sample spread across injection order is reduced for features labelled Drift_corrected
- Features with fewer than 4 QC samples carry the Missing_QCs note rather than a fitted correction
- No negative abundance values remain among features that are not flagged Negative_DC after back-transformation

## Limitations

- The notame package API is described as experimental and breaking changes are possible
- The original antonvsdata/notame repository is migrated and no longer maintained; active development is at hanhineva-lab/notame
- Features with fewer than 4 QC samples cannot be drift corrected and are recorded as Missing_QCs; corrections producing negative values are flagged Negative_DC

## Evidence

- [methods] Drift correction method: "Untargeted LC-MS metabolomics data usually suffers from signal intensity drift"
- [methods] spline function used: "The R function \code{smooth.spline} performs the modelling"
- [methods] correct_drift entry point: "To run drift correction on your data, use the function ```correct_drift```."
- [methods] mark_nas preprocessing: "```mark_nas``` replaces given values in ```exprs``` part of the MetaboSet object with ```NA```."
- [readme] drift correction feature in README: [evidence span withheld — anchor status: external_doc]
- [intro] scope of package: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
