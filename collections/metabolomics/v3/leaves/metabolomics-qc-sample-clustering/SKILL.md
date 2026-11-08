---
name: metabolomics-qc-sample-clustering
description: Use when a post-data-curation LC–MS feature matrix (e.g. from MetaboLights
  MTBLS1919) with pooled/intrastudy QC injections is available and you need to check
  QC clustering, sample outliers, or drift-correlated variance via PCA scores.
when_to_use_negative:
- Input is raw centroided/vendor LC–MS data rather than an extracted, curated feature
  matrix — run the TidyMS data-curation filters (Rt > 90 s, blank 10-fold, RSD 20%,
  D-ratio) first
- The goal is feature annotation or compound identification — the article notes TidyMS
  lacks feature annotation capability, and PCA scores do not provide it
- Study design has no pooled/intrastudy QC injections, so QC-based drift evaluation
  and the LOESS batch-correction precondition cannot be met
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: scikit-learn
  role: Provides the PCA implementation used to build the score-space model, per the
    paper's statistical analysis section
- name: TidyMS
  role: Supplies data-container conventions and the upstream curation pipeline whose
    filters (RSD 20%, blank, D-ratio) produce the matrix the PCA model inspects
  repo: https://github.com/griquelme/tidyms
- name: Pandas
  role: Loads the curated feature matrix from the MTBLS1919 deposit and writes the
    sample-by-component scores table
- name: Python
  role: Language in which the PCA-based QC inspection workflow is implemented and
    integrated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:80756735aabc0bbaf3467dad74ba9bc7ab9e1b507fb6ad0f2e843bbe1e88fd41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/griquelme/tidyms
  tools:
  - Pandas
  - Python
  - TidyMS
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# metabolomics-qc-sample-clustering

## Summary

Assesses LC–MS untargeted metabolomics data quality by fitting PCA models (scikit-learn) on the curated feature matrix from the NIST candidate RM 8231 / SRM 1950 study and inspecting score plots of pooled/intrastudy QC injections versus study samples, mirroring the paper's Figure 4 and Figure S3 before/after-curation comparison.

## When to use

Use when a post-data-curation LC–MS feature matrix (e.g. from MetaboLights MTBLS1919) with pooled/intrastudy QC injections is available and you need to check QC clustering, sample outliers, or drift-correlated variance via PCA scores.

## When NOT to use

- Input is raw centroided/vendor LC–MS data rather than an extracted, curated feature matrix — run the TidyMS data-curation filters (Rt > 90 s, blank 10-fold, RSD 20%, D-ratio) first
- The goal is feature annotation or compound identification — the article notes TidyMS lacks feature annotation capability, and PCA scores do not provide it
- Study design has no pooled/intrastudy QC injections, so QC-based drift evaluation and the LOESS batch-correction precondition cannot be met

## Inputs

- Curated (post data-curation) sample-by-feature matrix for NIST candidate RM 8231 and SRM 1950 plasma samples from MetaboLights MTBLS1919 (pandas-readable table)
- Sample metadata mapping injections to sample class (pooled/intrastudy QC, hypertriglyceridemic, diabetic, African-American plasma pools) and acquisition order/time

## Outputs

- Sample-by-PCA-component scores table (pca_scores.csv)
- PCA score plots of pooled/intrastudy QC and study samples before and after data curation (Figure 4 / Figure S3 style)

## How to apply

Load the curated (post data-curation) feature data matrix for the candidate RM 8231 and SRM 1950 plasma samples from the MetaboLights MTBLS1919 deposit using pandas via TidyMS data-container conventions. Fit a PCA model with the scikit-learn implementation on the cleaned matrix, then project pooled/intrastudy QC injections and study samples into score space to obtain sample scores per component. Use the scores to track data quality, flag outliers, reveal clusters of the hypertriglyceridemic, diabetic, and African-American plasma pools, and evaluate time-related systematic variance or signal drift. Generate score plots for models built on raw and curated data (mirroring Figure 4 and the Figure S3 comparison); the curated model should show tighter pooled-QC grouping without drift-correlated scores. Remember that the curation pipeline already removed features with RSD larger than 20% in pooled QC injections, applied a 10-fold blank filter, and used D-ratio filtering, so QC tightness in the curated model reflects these thresholds.

## Related tools

- **scikit-learn** (Provides the PCA implementation used to build the score-space model, per the paper's statistical analysis section)
- **TidyMS** (Supplies data-container conventions and the upstream curation pipeline whose filters (RSD 20%, blank, D-ratio) produce the matrix the PCA model inspects) — https://github.com/griquelme/tidyms
- **Pandas** (Loads the curated feature matrix from the MTBLS1919 deposit and writes the sample-by-component scores table)
- **Python** (Language in which the PCA-based QC inspection workflow is implemented and integrated)

## Examples

```
from sklearn.decomposition import PCA; import pandas as pd; scores = PCA(n_components=2).fit_transform(pd.read_csv('curated_matrix.csv')); scores.to_csv('pca_scores.csv')
```

## Evaluation signals

- Curated-data PCA model shows tighter pooled/intrastudy QC grouping than the raw-data model, reproducing the Figure S3 comparison
- QC scores show no correlation with injection time/acquisition order (no drift-correlated scores) after curation
- Score plots qualitatively separate the hypertriglyceridemic, diabetic, and African-American plasma pool clusters of candidate RM 8231
- pca_scores.csv is a well-formed sample-by-component table with one row per injected sample and columns per retained principal component
- Outliers flagged in score space are consistent with the QC/outlier pattern reported for the pooled QC models

## Limitations

- No software version numbers are stated for TidyMS, Python, or scikit-learn, affecting reproducibility of the PCA model
- TidyMS lacks feature annotation capability, so PCA-based QC can flag outliers and clusters but cannot explain their chemical origin
- Validation is qualitative: acceptance rests on reproducing the Figure S3 clustering/outlier pattern, not on a quantitative goodness-of-fit metric
- PCA on the curated matrix is meaningful only if upstream filters were applied consistently (e.g. features absent from pooled QC discarded as a precondition for LOESS batch correction)

## Evidence

- [results] Each step of the data curation process can be visually inspected by means of PCA models, as exemplified in Figure 4 with score plots displayed for the raw and curated data sets.: "Each step of the data curation process can be visually inspected by means of PCA models, as exemplified in Figure 4 with score plots"
- [results] all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
- [discussion] The implementation in Python allows easy integration of results into state-of-the-art tools for model generation, such as scikit-learn: "The implementation in Python allows easy integration of results into state-of-the-art tools for model generation, such as scikit-learn"
- [methods] The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919.: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
