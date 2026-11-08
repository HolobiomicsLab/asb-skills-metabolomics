---
name: log-space-abundance-back-transformation
description: 'Use when applying notame''s correct_drift() to LC-MS feature abundances:
  you log-transform the exprs matrix, fit a smooth.spline to QC samples across injection
  order, correct in log space, then must convert the corrected matrix back to linear
  intensities for output.'
when_to_use_negative:
- The abundance data have already been back-transformed to linear scale — exponentiating
  again would distort values
- Drift correction is being skipped (e.g. the dataset has too few QC samples, fewer
  than 4, and correct_drift() records Missing_QCs) — there is no log-space corrected
  matrix to transform back
- The data are not log-transformed LC-MS feature abundances (e.g. peak areas on another
  scale) and the additive log-space correction model does not apply
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: Provides correct_drift(), which performs the log-space spline correction and
    back-transforms the corrected abundances to linear scale before writing the result
  repo: https://github.com/hanhineva-lab/notame
- name: smooth.spline
  role: R function used to fit the drift model to QC samples in log space; the back-transformation
    is the inverse of the log step taken before this fit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:d0585aab844072b6ff40194f1ee438ff9f6abcdd57bfc402bace8a5b0ad527ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  - smooth.spline
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# log-space-abundance-back-transformation

## Summary

The back-transformation step of notame's drift correction: after fitting a cubic spline to log-transformed QC abundances versus injection order, corrected log-space values are exponentiated to return the feature abundance matrix to linear scale, satisfying the spline model's assumptions while delivering results in the original measurement units.

## When to use

Use when applying notame's correct_drift() to LC-MS feature abundances: you log-transform the exprs matrix, fit a smooth.spline to QC samples across injection order, correct in log space, then must convert the corrected matrix back to linear intensities for output.

## When NOT to use

- The abundance data have already been back-transformed to linear scale — exponentiating again would distort values
- Drift correction is being skipped (e.g. the dataset has too few QC samples, fewer than 4, and correct_drift() records Missing_QCs) — there is no log-space corrected matrix to transform back
- The data are not log-transformed LC-MS feature abundances (e.g. peak areas on another scale) and the additive log-space correction model does not apply

## Inputs

- notame MetaboSet object (or exprs matrix) with log-transformed drift-corrected feature abundances from QC and biological samples, ordered by injection order
- Per-feature DC_note quality labels produced by correct_drift() (Drift_corrected, Low-quality, Missing_QCs, Negative_DC)

## Outputs

- Feature abundance matrix in original (linear) intensity scale, same samples × features dimensions as the input log-space matrix
- Drift-corrected abundance table suitable for writing to drift_corrected_exprs.csv with accompanying DC_note column

## How to apply

In the notame workflow, drift correction is run on log-transformed abundances so the cubic spline (R's smooth.spline, fitted to QC samples only, with the smoothing parameter chosen by leave-one-out cross-validation in the recommended 0.5–1.5 interval) models additive drift in log space. After computing the correction x_corrected(i) = x_original(i) + mean_QC − f_drift(i) for every sample, apply the inverse transformation by exponentiating (e^x) each corrected value to return the abundance matrix to linear scale. Check the per-feature DC_note column: features marked Drift_corrected are usable, while Negative_DC flags cases where back-transformation produced negative values, and Missing_QCs indicates fewer than 4 QC samples prevented spline fitting. The result is a corrected exprs matrix in linear units, suitable for downstream flagging, imputation, and batch correction steps in notame.

## Related tools

- **notame** (Provides correct_drift(), which performs the log-space spline correction and back-transforms the corrected abundances to linear scale before writing the result) — https://github.com/hanhineva-lab/notame
- **smooth.spline** (R function used to fit the drift model to QC samples in log space; the back-transformation is the inverse of the log step taken before this fit)

## Examples

```
corrected <- correct_drift(set); linear_corrected <- 2^assay(corrected, "exprs")
```

## Evaluation signals

- The output matrix has the same samples × features dimensions as the input and no negative values except where DC_note = Negative_DC
- Features labelled Drift_corrected show reduced spread of QC-sample abundances across injection order compared with the pre-correction data
- The DC_note column is present with values drawn from {Drift_corrected, Low-quality, Missing_QCs, Negative_DC}
- QC samples whose corrected values match their fitted drift level should have abundances close to the original QC mean after back-transformation
- Re-logging the back-transformed values reproduces the log-space corrected matrix (identity check: log(exp(x)) = x, up to floating-point precision)

## Limitations

- Negative values can arise after correction; notame flags these with DC_note = Negative_DC, which is a limitation of applying the additive log-space correction to weak signals
- Back-transformation is only meaningful when the spline was fitted in log space — applying it to linear-scale corrections is a common misuse
- Features with insufficient QC samples (< 4) are labelled Missing_QCs and cannot be reliably corrected or back-transformed

## Evidence

- [methods] correct_drift() log-transforms each feature, fits a smoothed cubic spline to QC samples to model drift as a function of injection order: [evidence span withheld — anchor status: not_found]
- [methods] notame runs drift correction on log-transformed data to satisfy the spline model assumptions: [evidence span withheld — anchor status: not_found]
- [methods] The back-transformation formula and DC_note flags: [evidence span withheld — anchor status: not_found]
- [methods] Workflow validation of the corrected output: [evidence span withheld — anchor status: not_found]
- [readme] README statement that drift correction uses cubic spline correction: [evidence span withheld — anchor status: external_doc]
