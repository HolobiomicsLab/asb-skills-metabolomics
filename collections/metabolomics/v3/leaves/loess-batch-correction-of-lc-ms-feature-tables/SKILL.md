---
name: loess-batch-correction-of-lc-ms-feature-tables
description: Use when a QC-filtered LC-MS feature table contains pooled/intrastudy
  QC injections in run order and time-related signal drift must be removed from study-sample
  intensities before normalization or statistical analysis.
when_to_use_negative:
- The feature table still contains features absent from pooled/intrastudy QC samples
  — the article's filter step discards these first as a condition for LOESS batch
  correction.
- Upstream QC curation has not been done (e.g., features with RSD larger than 20%
  in pooled/intrastudy QC injections not yet eliminated) — correction on uncurated
  features will fit to unreliable intensities.
- Input is raw LC-MS data rather than an extracted feature matrix from feature detection
  and correspondence — TidyMS's data curation pipeline, not this correction step,
  comes first.
tools:
- name: TidyMS
  role: Python package hosting the Data Container for the feature table and the LOESS
    batch correction step of the data curation pipeline
  repo: https://github.com/griquelme/tidyms
- name: statsmodels
  role: Provides the LOESS smoother implementation fitted per feature to pooled QC
    injection intensities
- name: scikit-learn
  role: Provides the PCA model of pooled QC samples used to evaluate reduction of
    time-related systematic variance after correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:584e4a717febe6e6fdffa12fd83d0eb1f55da3b177ce187e73635ea8b841fbb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - TidyMS
  - scikit-learn
  - statsmodels
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# LOESS batch correction of LC-MS feature tables

## Summary

Per-feature LOESS smoother fitted to pooled/intrastudy QC injections removes time-related systematic variance (signal drift) from a TidyMS LC-MS feature table, with the LOESS fraction optimized per feature by LOOCV.

## When to use

Use when a QC-filtered LC-MS feature table contains pooled/intrastudy QC injections in run order and time-related signal drift must be removed from study-sample intensities before normalization or statistical analysis.

## When NOT to use

- The feature table still contains features absent from pooled/intrastudy QC samples — the article's filter step discards these first as a condition for LOESS batch correction.
- Upstream QC curation has not been done (e.g., features with RSD larger than 20% in pooled/intrastudy QC injections not yet eliminated) — correction on uncurated features will fit to unreliable intensities.
- Input is raw LC-MS data rather than an extracted feature matrix from feature detection and correspondence — TidyMS's data curation pipeline, not this correction step, comes first.

## Inputs

- QC-filtered feature table (retention time, m/z feature matrix, CSV) loaded into a TidyMS Data Container with pandas backing
- Sample-type mapping associating study samples, blanks, and pooled/intrastudy QC injections with their run order

## Outputs

- batch_corrected_data_matrix.csv containing all LOESS-corrected features and samples
- PCA model (scikit-learn) of pooled QC samples before and after correction for qualitative drift-reduction assessment

## How to apply

Load the QC-filtered feature table into a TidyMS Data Container with the sample mapping that marks pooled/intrastudy QC injections and their injection order. Restrict correction to features present in pooled/intrastudy QC samples, as required by the paper's Supporting Text 1 condition for performing LOESS batch correction. For each retained feature, fit a LOESS smoother (statsmodels implementation) to the feature intensities across the ordered pooled QC injections; by default the fraction of samples used by the LOESS approach is optimized per feature via leave-one-out cross-validation (LOOCV). Apply the fitted smoothers to remove signal drift from study-sample intensities and assemble the corrected matrix, then save it as batch_corrected_data_matrix.csv.

## Related tools

- **TidyMS** (Python package hosting the Data Container for the feature table and the LOESS batch correction step of the data curation pipeline) — https://github.com/griquelme/tidyms
- **statsmodels** (Provides the LOESS smoother implementation fitted per feature to pooled QC injection intensities)
- **scikit-learn** (Provides the PCA model of pooled QC samples used to evaluate reduction of time-related systematic variance after correction)

## Examples

```
from statsmodels.nonparametric.smoothers_lowess import lowess; corrected = lowess(qc_intensity, injection_order, frac=frac_loocv, return_sorted=False)
```

## Evaluation signals

- batch_corrected_data_matrix.csv exists as a CSV containing all LOESS-corrected features and samples.
- All corrected intensity values in the output matrix are finite (no NaN/inf introduced by the smoother).
- PCA (scikit-learn) of the corrected pooled QC injections shows reduced time-related signal drift, qualitatively mirroring the paper's before/after pooled-QC PCA comparison.
- Every corrected feature was present in the pooled/intrastudy QC injections (the Supporting Text 1 condition for LOESS).
- Per-feature LOESS fraction parameters reflect LOOCV optimization rather than a single arbitrary fraction.

## Limitations

- Features not present in pooled/intrastudy QC samples cannot be LOESS-corrected and are discarded by the preceding filter step.
- Correction quality is assessed qualitatively (before/after pooled-QC PCA) rather than by a numeric acceptance threshold.
- No software version numbers are stated for TidyMS, Python, or scikit-learn in the article, so exact reproducibility of the LOESS configuration requires consultation of the tidyms-notebooks examples.

## Evidence

- [results] Features must be present in pooled QC as precondition for LOESS: "The second step allowed discarding features that were not present in pooled /intrastudy QC samples (see Supporting Text 1) as a condition for performing the LOESS batch correction"
- [results] Upstream QC curation precedes correction (RSD 20% filter): "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [intro] Normalization/drift handling influences downstream statistics: "data normalization strategies implemented before processing data and performing statistical analysis can dramatically influence results"
