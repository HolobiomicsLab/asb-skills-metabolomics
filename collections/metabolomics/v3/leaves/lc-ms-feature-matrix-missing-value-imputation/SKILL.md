---
name: lc-ms-feature-matrix-missing-value-imputation
description: Use when a peak-intensity profile table from LC-MS or SRM-MS metabolomics/proteomics
  contains missing values that must be filled before applying the 80% rule-based peak
  filter, QC-RFSC drift correction, or multivariate statistics in statTarget.
when_to_use_negative:
- The input is already a fully populated feature matrix with no missing values — imputation
  would be a no-op and could distort real low-abundance signals.
- The dataset contains no QC injections (e.g. QC-free studies) — use shiftCor_dQC
  with MLmethod = 'Combat' instead of the QC-RFSC pipeline, though KNN imputation
  via imputeM still applies in statAnalysis.
- Missingness exceeds what the 80% rule (Frule = 0.8) permits — features failing the
  filter are removed before imputation and should not be force-imputed.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: statTarget
  role: Hosts the shiftCor and statAnalysis functions that expose the KNN imputation
    step (imputeM = 'KNN') for feature-intensity matrices
  repo: https://github.com/13479776/statTarget
- name: shiftCor
  role: Applies the 80% rule filter then KNN imputation before fitting the QC-RFSC
    random forest correction model
  repo: https://github.com/13479776/statTarget
- name: statAnalysis
  role: Alternative entry point that imputes missing values with imputeM = 'KNN' when
    only statistical analysis (normalization, glog, Pareto scaling) is needed
  repo: https://github.com/13479776/statTarget
- name: transX
  role: Converts XCMS/MZmine2/SIEVE/Skyline peak tables into the statTarget input
    format so the imputation step can operate on the profile file
  repo: https://github.com/13479776/statTarget
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:e2fbb57c0e9bd4cf94b721aef0eaebeecff8b0df4f259ba891c172384b8ed4d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/13479776/statTarget
  tools:
  - shiftCor
  - statAnalysis
  - statTarget
  - transX
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# LC-MS feature matrix missing value imputation

## Summary

Impute missing peak-intensity values in LC-MS feature matrices (XCMS/MZmine2/SIEVE/Skyline exports) with K-nearest-neighbor averaging, as implemented in the statTarget shiftCor and statAnalysis functions, prior to QC-RFSC signal drift correction and downstream statistical analysis.

## When to use

Use when a peak-intensity profile table from LC-MS or SRM-MS metabolomics/proteomics contains missing values that must be filled before applying the 80% rule-based peak filter, QC-RFSC drift correction, or multivariate statistics in statTarget.

## When NOT to use

- The input is already a fully populated feature matrix with no missing values — imputation would be a no-op and could distort real low-abundance signals.
- The dataset contains no QC injections (e.g. QC-free studies) — use shiftCor_dQC with MLmethod = 'Combat' instead of the QC-RFSC pipeline, though KNN imputation via imputeM still applies in statAnalysis.
- Missingness exceeds what the 80% rule (Frule = 0.8) permits — features failing the filter are removed before imputation and should not be force-imputed.

## Inputs

- Feature-by-sample peak-intensity profile table (statTarget samFile format, converted from XCMS/MZmine2/SIEVE/Skyline output via transX)
- Sample meta file (samPeno) with sample name, class, batch, and injection order
- Numeric parameter Frule = 0.8 governing the pre-imputation 80% rule filter

## Outputs

- Complete (imputed) feature-intensity matrix ready for QC-RFSC correction or statistical analysis
- Corrected matrices (all samples, QC only, samples only) written by shiftCor after imputation and correction-factor division
- RSD/CV distribution tables and per-feature loplot QC images for raw versus corrected peaks

## How to apply

Load the profile file (feature × sample peak intensities exported from XCMS, MZmine2, SIEVE, or Skyline via statTarget's transX) together with the meta file holding sample name, class, batch, and order. Apply the modified 80 percent rule first (Frule = 0.8), keeping only features with non-zero values in at least 80% of samples in any one group, since the QC-RFSC algorithm cannot tolerate excessive missingness. Then impute the remaining missing values with the nearest-neighbor averaging method by setting imputeM = 'KNN' in either shiftCor (for QC-based drift correction) or statAnalysis (for the statistical workflow). After correction, verify imputation quality by checking that QC-peak RSD distributions improve (e.g. in dataset 1 the proportion of QC peaks with RSD < 30% rose from about 43.9% raw to about 86.2% corrected).

## Related tools

- **statTarget** (Hosts the shiftCor and statAnalysis functions that expose the KNN imputation step (imputeM = 'KNN') for feature-intensity matrices) — https://github.com/13479776/statTarget
- **shiftCor** (Applies the 80% rule filter then KNN imputation before fitting the QC-RFSC random forest correction model) — https://github.com/13479776/statTarget
- **statAnalysis** (Alternative entry point that imputes missing values with imputeM = 'KNN' when only statistical analysis (normalization, glog, Pareto scaling) is needed) — https://github.com/13479776/statTarget
- **transX** (Converts XCMS/MZmine2/SIEVE/Skyline peak tables into the statTarget input format so the imputation step can operate on the profile file) — https://github.com/13479776/statTarget

## Examples

```
shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN")
```

## Evaluation signals

- After filtering with Frule = 0.8, no remaining feature should have a missing value in more than 20% of samples within any one group before imputation.
- The imputed matrix should have zero missing entries across all features and samples, matching the complete matrix shape expected by QC-RFSC.
- Compare QC-peak RSD distributions before and after correction: a successful pipeline increases the share of QC peaks with RSD < 30% (article reports 43.9% → 86.2% in dataset 1, and 52.1% → 90.9% in dataset 2).
- Imputed values should not create artificial batch effects — the correction factor Fij applied per feature and injection order should still yield improved RSD within each batch and overall.
- Output files (corrected matrices for all samples, QCs only, samples only) should contain no NaN/zero-inflated rows that would indicate failed imputation.

## Limitations

- KNN imputation is performed only on features that survive the 80% rule (Frule = 0.8); low-quality peaks removed by the filter are lost and cannot be recovered.
- The imputation step in shiftCor assumes QC samples are identifiable by sample names tagged 'QC' — mislabelled or absent QCs break the QC-RFSC correction that follows imputation.
- Imputation quality depends on the underlying batch/injection-order structure; uncorrected inter- and intra-batch drift can bias nearest-neighbor distances if not handled by QC-RFSC.
- Other correction methods benchmarked in the article (QC-RLSC, QC-SVR) achieved lower RSD improvements, so imputation paired with QC-RFSC is preferred over simpler correction pipelines.

## Evidence

- [results] RSD improvement benchmark used to validate correction after imputation: [evidence span withheld — anchor status: not_found]
