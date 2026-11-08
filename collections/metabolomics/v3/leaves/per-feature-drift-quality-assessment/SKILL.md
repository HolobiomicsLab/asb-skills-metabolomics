---
name: per-feature-drift-quality-assessment
description: Use when a preprocessed LC-MS feature abundance matrix contains QC samples
  spread across injection order and features show systematic intensity drift that
  must be corrected and assessed per feature before imputation or statistics.
when_to_use_negative:
- Data has not yet been peak-picked into a feature table; notame operates on preprocessed
  LC-MS output, not raw instrument files.
- Injection order is not randomized or no repeated QC samples are available, so no
  drift signal can be estimated.
- The analysis goal is between-batch variation, not within-run drift; use batch effect
  correction strategies instead.
edam_operation: http://edamontology.org/operation_3287
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: notame
  role: Provides the MetaboSet object, correct_drift function, and DC_note flagging
    used for per-feature drift correction and quality assessment
  repo: https://github.com/hanhineva-lab/notame
- name: smooth.spline
  role: R function used internally to fit the cubic spline model of drift as a function
    of injection order for each feature
- name: MS-DIAL
  role: Peak picking software that produces the input Excel peak table consumed by
    this skill
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:86295c01040d6ddebc9e73cbd8d5b90388822cdb5788620986a651472b6aa6fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - MS-DIAL
  - R
  - notame
  - smooth.spline
  edam_operation: http://edamontology.org/operation_3287
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# per-feature-drift-quality-assessment

## Summary

Systematic signal intensity drift in untargeted LC-MS metabolomics is corrected per molecular feature with a cubic-spline model fitted to QC samples, and each feature is flagged with a DC_note recording whether correction succeeded, failed, or degraded quality.

## When to use

Use when a preprocessed LC-MS feature abundance matrix contains QC samples spread across injection order and features show systematic intensity drift that must be corrected and assessed per feature before imputation or statistics.

## When NOT to use

- Data has not yet been peak-picked into a feature table; notame operates on preprocessed LC-MS output, not raw instrument files.
- Injection order is not randomized or no repeated QC samples are available, so no drift signal can be estimated.
- The analysis goal is between-batch variation, not within-run drift; use batch effect correction strategies instead.

## Inputs

- MS-DIAL Excel peak table (with corner row/column annotations and Column and Ion mode split columns)
- MetaboSet object with exprs feature abundance matrix, sample metadata including injection order and QC group labels
- Integer vector or column identifying injection order per sample

## Outputs

- Drift-corrected abundance matrix (back-transformed to linear scale)
- Per-feature DC_note column with values Drift_corrected, Low-quality, Missing_QCs, or Negative_DC
- Plots of fitted spline correction curves per feature for visual QC

## How to apply

Load the MS-DIAL peak table into a MetaboSet with notame::read_from_excel (corner_row=4, corner_column="X", split_by=c("Column","Ion mode")) and construct_metabosets with the group column, then mark zero abundances as missing via mark_nas. Log-transform the exprs matrix because drift correction is fitted in log space to satisfy the spline model assumptions. For each feature, fit smooth.spline to the QC samples' log-abundance versus injection order; the smoothing parameter is chosen by cross-validation within the recommended interval 0.5-1.5, and at least 4 QC samples are required, otherwise the feature is recorded as Missing_QCs. Correct each sample additively as x_corrected(i) = x_original(i) + mean_QC - f_drift(i) in log space, back-transform to linear scale, and assign a DC_note of Drift_corrected, Low-quality, Missing_QCs, or Negative_DC depending on whether quality improved, QCs were insufficient, or negative values arose. Inspect the plotted correction curves and QC spread across injection order to judge whether correction genuinely reduced drift rather than degrading signal.

## Related tools

- **notame** (Provides the MetaboSet object, correct_drift function, and DC_note flagging used for per-feature drift correction and quality assessment) — https://github.com/hanhineva-lab/notame
- **smooth.spline** (R function used internally to fit the cubic spline model of drift as a function of injection order for each feature)
- **MS-DIAL** (Peak picking software that produces the input Excel peak table consumed by this skill)

## Examples

```
corrected <- correct_drift(object, log_transform = TRUE); write_result(corrected, "drift_corrected_exprs.csv")
```

## Evaluation signals

- drift_corrected_exprs.csv exists with one row per sample and one column per feature, plus a DC_note column
- QC-sample spread of log-abundance across injection order is visibly reduced for features labelled Drift_corrected
- No feature receives a corrected value that is negative without a corresponding Negative_DC DC_note
- Features with fewer than 4 QC observations are flagged Missing_QCs and left uncorrected rather than silently imputed
- The distribution of DC_note categories is reported (counts of Drift_corrected vs Low-quality vs Negative_DC) before proceeding to imputation

## Limitations

- Requires at least 4 QC samples per feature; features with insufficient QC observations are marked Missing_QCs and not corrected.
- Correction is performed in log space, so negative corrected values can appear after back-transformation and must be flagged as Negative_DC.
- The package API is still experimental and breaking changes are possible, so function signatures may shift between versions.
- Low-quality features flagged by quality metrics may still be retained after correction and should be removed or flagged before downstream statistics.

## Evidence

- [methods] Drift correction: "Untargeted LC-MS metabolomics data usually suffers from signal intensity drift"
- [readme] notame description: [evidence span withheld — anchor status: external_doc]
- [readme] notame features: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics."
- [methods] spline function: "The R function \code{smooth.spline} performs the modelling"
- [methods] workflow step: "As a first step in data preprocessing, missing values should be correctly represented."
