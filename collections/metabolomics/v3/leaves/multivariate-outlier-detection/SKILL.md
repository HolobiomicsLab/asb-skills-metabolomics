---
name: multivariate-outlier-detection
description: Use when a post-curation feature data matrix plus pooled/intrastudy QC
  injections are available and you need to verify curation quality, spot outliers,
  QC clustering, or time-related signal drift before statistical analysis.
when_to_use_negative:
- Input is still the raw extracted matrix before the TidyMS curation filters (Rt filter,
  blank filter, RSD filter, prevalence filter, D-ratio filter) have been applied,
  when the goal is to judge curation outcome rather than raw-data condition
- The dataset lacks pooled/intrastudy QC injections, so no QC-referenced quality or
  drift assessment can be anchored in score space
- The goal is feature annotation or biological group-difference testing rather than
  QC/outlier inspection of the data matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: PCA implementation used to build the models and project QC injections and
    study samples into score space
- name: TidyMS
  role: Data-container conventions for loading the curated feature matrix prior to
    PCA
  repo: https://github.com/griquelme/tidyms
- name: Pandas
  role: Loading the curated feature matrix and writing the sample-by-component scores
    table to pca_scores.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:d217574894c255f6c901f002b80c82f9c3c3e3268ba86d35d985393ff672c481
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Pandas
  - TidyMS
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multivariate-outlier-detection

## Summary

Build PCA models with scikit-learn on raw and curated LC-MS feature matrices (NIST candidate RM 8231 and SRM 1950 plasma, MetaboLights MTBLS1919) and inspect score plots of pooled/intrastudy QC injections to visually assess data quality, outliers, and drift at each step of the TidyMS data-curation pipeline.

## When to use

Use when a post-curation feature data matrix plus pooled/intrastudy QC injections are available and you need to verify curation quality, spot outliers, QC clustering, or time-related signal drift before statistical analysis.

## When NOT to use

- Input is still the raw extracted matrix before the TidyMS curation filters (Rt filter, blank filter, RSD filter, prevalence filter, D-ratio filter) have been applied, when the goal is to judge curation outcome rather than raw-data condition
- The dataset lacks pooled/intrastudy QC injections, so no QC-referenced quality or drift assessment can be anchored in score space
- The goal is feature annotation or biological group-difference testing rather than QC/outlier inspection of the data matrix

## Inputs

- Curated (post data-curation) feature data matrix (sample x feature) for candidate RM 8231 and SRM 1950 plasma samples from MetaboLights MTBLS1919
- Sample metadata table labeling pooled/intrastudy QC injections, study samples, and injection order/time for drift assessment
- Optionally, the raw (pre-curation) feature matrix for before/after score-plot comparison

## Outputs

- Sample-by-component PCA scores table written to pca_scores.csv
- PCA score plots of pooled QC and study samples for raw and curated data sets (Figure 4 / Figure S3 style)

## How to apply

Load the curated feature data matrix for the candidate RM 8231 and SRM 1950 plasma samples from the MetaboLights MTBLS1919 deposit using pandas via TidyMS data-container conventions. Fit a PCA model using the scikit-learn implementation, as stated in the paper's statistical analysis section, on the cleaned data matrix. Project pooled/intrastudy QC injections and study samples into the PCA score space and inspect score plots before and after curation (mirroring Figure S3 and Figure 4 of the paper). Judge success qualitatively: after curation (Rt > 90 s, blank 10-fold filter, RSD < 20% in pooled QC, 100% intraclass prevalence with area threshold 5, D-ratio filter) pooled-QC scores should group more tightly and lose drift-correlated structure, while study-sample clusters (hypertriglyceridemic, diabetic, and African-American plasma pools) should remain distinguishable.

## Related tools

- **scikit-learn** (PCA implementation used to build the models and project QC injections and study samples into score space)
- **TidyMS** (Data-container conventions for loading the curated feature matrix prior to PCA) — https://github.com/griquelme/tidyms
- **Pandas** (Loading the curated feature matrix and writing the sample-by-component scores table to pca_scores.csv)

## Examples

```
import pandas as pd; from sklearn.decomposition import PCA; X = pd.read_csv('curated_matrix.csv', index_col=0); scores = PCA(n_components=2).fit_transform(X); scores.to_csv('pca_scores.csv')
```

## Evaluation signals

- Curated-data PCA model shows tighter pooled/intrastudy QC grouping than the raw-data model (Figure S3 comparison)
- Curated-data QC scores show no time/drift-correlated ordering, unlike the raw-data model
- Study-sample clusters (hypertriglyceridemic, diabetic, and African-American plasma pools) are qualitatively reproducible in the curated score plots
- PCA scores table (pca_scores.csv) has one row per sample (QC plus study samples) with the fitted component columns
- No obvious outlier samples in the curated model that are not traceable to a failed curation filter step

## Limitations

- PCA-based QC assessment is a qualitative visual inspection of score plots, not a quantitative pass/fail test
- QC quality judgement depends on the preceding curation thresholds (e.g. RSD < 20% in pooled QC, D-ratio filter, 10-fold blank filter); errors there propagate into the PCA interpretation
- No software version numbers for TidyMS, Python, or scikit-learn are stated in the paper, so exact reproducibility of the score space may vary

## Evidence

- [results] Each step of the data curation process can be visually inspected by means of PCA models, as exemplified in Figure 4 with score plots displayed for the raw and curated data sets.: [evidence span withheld — anchor status: not_found]
- [intro] Python packages include widely used machine learning frameworks, such as scikit-learn: "Python packages include widely used machine learning frameworks, such as scikit-learn [27] and Tensor Flow [28],"
- [results] all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [methods] candidate RM 8231 Frozen Human Plasma Suite for Metabolomics (hypertriglyceridemic, diabetic, and African-American plasma samples): "candidate RM 8231 Frozen Human Plasma Suite for Metabolomics (hypertriglyceridemic, diabetic, and African-American plasma samples)"
