---
name: pca-data-quality-tracking
description: Use when a pooled/intrastudy QC-injected LC-MS batch (e.g. NIST candidate
  RM 8231 / SRM 1950 plasma in mzML with a Pandas feature table) must be checked for
  drift, outliers, or group separation via PCA score plots.
when_to_use_negative:
- Input is raw mzML signal rather than a feature table - run TidyMS peak picking (CWT),
  alignment, deisotoping, and correspondence first
- You need feature annotation or identity assignment - the article reports TidyMS
  lacks annotation capability, and PCA scores alone do not annotate features
- The dataset contains no pooled/intrastudy QC injections - drift evaluation and the
  QC-centric PCA assessment of this paper cannot be performed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TidyMS
  role: Loads and hosts the curated feature data matrix (Data Container on Pandas)
    from the MTBLS1919 mzML-derived data for PCA-based QC tracking
  repo: https://github.com/griquelme/tidyms
- name: scikit-learn
  role: Provides the PCA model implementation used to project QC and study samples
    into score space for drift and outlier assessment
- name: Pandas
  role: Data-frame layer of the TidyMS Data Container used to assemble and export
    the sample-by-component scores table
provenance:
  source_task_ids:
  - task_008
  - task_003
  - task_004
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:558ff18f1379b31508ad11d5fa7bc310262cb5f56b51e76859f46895b372aae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Pandas
  - TidyMS
  - pyopenms
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pca-data-quality-tracking

## Summary

Build PCA models (scikit-learn) on the TidyMS-curated LC-MS feature matrix from the MTBLS1919 NIST plasma runs and inspect pooled/intrastudy QC score plots to detect signal drift, outliers, and clustering before and after data curation.

## When to use

Use when a pooled/intrastudy QC-injected LC-MS batch (e.g. NIST candidate RM 8231 / SRM 1950 plasma in mzML with a Pandas feature table) must be checked for drift, outliers, or group separation via PCA score plots.

## When NOT to use

- Input is raw mzML signal rather than a feature table - run TidyMS peak picking (CWT), alignment, deisotoping, and correspondence first
- You need feature annotation or identity assignment - the article reports TidyMS lacks annotation capability, and PCA scores alone do not annotate features
- The dataset contains no pooled/intrastudy QC injections - drift evaluation and the QC-centric PCA assessment of this paper cannot be performed

## Inputs

- TidyMS-curated LC-MS feature data matrix (Pandas/TidyMS Data Container) for NIST candidate RM 8231 and SRM 1950 plasma samples from the MTBLS1919 deposit
- Sample metadata labeling pooled/intrastudy QC injections versus study samples, with acquisition order for drift tracking
- Optional: pre-curation (raw extracted) feature matrix for before/after comparison score plots

## Outputs

- PCA score plot figures for pooled/intrastudy QC and study samples before and after data curation (analogous to Figure 4 and Figure S3)
- Sample-by-PC scores table (e.g. pca_scores.csv) usable for drift, outlier, and cluster assessment

## How to apply

Load the post-curation feature matrix (features x samples) for candidate RM 8231 and SRM 1950 from the MetaboLights MTBLS1919 deposit using the TidyMS Data Container built on Pandas, tagging pooled/intrastudy QC injections in acquisition order. Fit a PCA model with the scikit-learn implementation (the package TidyMS relies on for model generation) and project both QC injections and study samples into score space. Inspect the score plots for time-related systematic variance or signal drift along PC1, outlier injections, and clusters such as the hypertriglyceridemic, diabetic, and African-American plasma pools. Generate score plots before and after curation (mirroring Figure 4 / Figure S3) to confirm that curation steps - Rt filter (>= 90 s), blank correction (10x rule), 20% QC RSD, prevalence, and D-ratio filters - tightened QC grouping and removed drift. Export the sample-by-component scores (e.g. pca_scores.csv) for downstream statistical analysis.

## Related tools

- **TidyMS** (Loads and hosts the curated feature data matrix (Data Container on Pandas) from the MTBLS1919 mzML-derived data for PCA-based QC tracking) — https://github.com/griquelme/tidyms
- **scikit-learn** (Provides the PCA model implementation used to project QC and study samples into score space for drift and outlier assessment)
- **Pandas** (Data-frame layer of the TidyMS Data Container used to assemble and export the sample-by-component scores table)

## Examples

```
from sklearn.decomposition import PCA; scores = PCA().fit_transform(X_curated); pd.DataFrame(scores, index=sample_ids).to_csv('pca_scores.csv')
```

## Evaluation signals

- Curated-data PCA scores show tighter pooled/intrastudy QC grouping than the raw-data model, matching the Figure S3 before/after comparison
- Drift-correlated scores along PC1 present in the pre-curation model are absent (or markedly reduced) after the LOESS batch correction step
- QC injections' scores reflect the acquisition order such that time-related systematic variance is visible in the raw model and quantifiable across PCs
- Pooled QC clusters of hypertriglyceridemic, diabetic, and African-American plasma reference material samples separate in score space consistent with the paper's reported models
- Sample scores are complete (one row per injected sample) and rank-consistent with the metadata acquisition order

## Limitations

- PCA is used here for QC assessment only; the article does not report TidyMS feature annotation, so scores cannot identify which metabolites drive separation
- Drift evaluation depends on pooled/intrastudy QC samples being injected in sequence; without them, drift along PC1 cannot be tracked
- No software version numbers are stated for TidyMS, Python, or scikit-learn, so reproductions should note the versions actually used

## Evidence

- [results] Each step of the data curation process can be visually inspected by means of PCA models, as exemplified in Figure 4 with score plots displayed for the raw and curated data sets.: [evidence span withheld — anchor status: not_found]
- [methods] Build a PCA model with the scikit-learn implementation (as stated in the paper's statistical analysis section) on the cleaned data matrix.: [evidence span withheld — anchor status: not_found]
- [results] PCA score plots built from pooled/intrastudy QC samples revealed the presence of signal drift along PC1, and this trend was removed after applying the LOESS batch correction during data curation.: [evidence span withheld — anchor status: not_found]
- [abstract] The package was used to preprocess data for candidate NIST plasma reference materials (hypertriglyceridemic, diabetic, and African-American plasma pools).: "preprocess data corresponding to a new suite of candidate plasma reference materials developed by the National Institute of Standards and Technology (NIST; hypertriglyceridemic, diabetic, and"
- [other] Use the scores to track data quality, identify potential outliers, reveal sample clusters (e.g. hypertriglyceridemic, diabetic, and African-American plasma pools), and evaluate time-related systematic variance or signal drift.: [evidence span withheld — anchor status: not_found]
