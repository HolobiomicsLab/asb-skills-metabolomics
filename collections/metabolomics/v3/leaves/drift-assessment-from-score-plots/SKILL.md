---
name: drift-assessment-from-score-plots
description: Use when you have a post-curation extracted feature data matrix from
  a LC–MS untargeted metabolomics run that includes pooled/intrastudy QC injections,
  and you need to verify data quality — outlier detection, QC clustering tightness,
  and detection of injection-order–related signal drift — using.
when_to_use_negative:
- The input is a raw, uncurated feature matrix where unwanted features (low retention
  time, blank-heavy, high-RSD) have not yet been removed — drift assessment on uncurated
  data will reflect noise rather than true batch correction quality.
- The analysis goal is feature-level annotation or metabolite identification; this
  skill only inspects sample-level score space and does not assign identities to retention
  time / m/z features.
- The dataset lacks pooled/intrastudy QC injections or comparable QC samples, making
  QC-based drift tracking impossible.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: scikit-learn
  role: Implements the PCA model whose score space is used to track data quality,
    outliers, clusters, and signal drift
- name: TidyMS
  role: Provides the curated feature data matrix (post data-curation) and data-container
    conventions used to load and organize samples and QC injections
  repo: https://github.com/griquelme/tidyms
- name: Pandas
  role: Loads and manipulates the extracted feature data matrix from the MetaboLights
    MTBLS1919 deposit before PCA fitting
- name: Python
  role: Programming language in which the PCA-based QC assessment pipeline is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo10100416
    title: A Python-Based Pipeline for Preprocessing LC–MS Data for Untargeted Metabolomics
      Workflows
  source_content_hash: sha256:0ffb07198dfe2f58ff0e9c5e8bb43f85b077331cb05e0acacbeff8856ff7f5e0
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# drift-assessment-from-score-plots

## Summary

Builds a PCA model on the curated LC–MS feature matrix (pooled/intrastudy QC injections plus study samples from the NIST candidate RM 8231 and SRM 1950 plasma datasets) and inspects score plots to reveal sample clusters, outliers, and time-related systematic variance or signal drift before and after data curation.

## When to use

Use when you have a post-curation extracted feature data matrix from a LC–MS untargeted metabolomics run that includes pooled/intrastudy QC injections, and you need to verify data quality — outlier detection, QC clustering tightness, and detection of injection-order–related signal drift — using PCA score space.

## When NOT to use

- The input is a raw, uncurated feature matrix where unwanted features (low retention time, blank-heavy, high-RSD) have not yet been removed — drift assessment on uncurated data will reflect noise rather than true batch correction quality.
- The analysis goal is feature-level annotation or metabolite identification; this skill only inspects sample-level score space and does not assign identities to retention time / m/z features.
- The dataset lacks pooled/intrastudy QC injections or comparable QC samples, making QC-based drift tracking impossible.

## Inputs

- Curated (post data-curation) LC–MS extracted feature data matrix (samples × features) from the candidate RM 8231 and SRM 1950 plasma samples (MetaboLights MTBLS1919)
- Sample metadata / run-order annotations distinguishing pooled/intrastudy QC injections from study samples
- Optionally, the raw (pre-curation) extracted feature matrix for the before-vs-after comparison

## Outputs

- PCA score plots for the raw and curated data sets (mirroring Figure 4 and Figure S3)
- Sample-by-principal-component scores table written to pca_scores.csv
- Qualitative assessment of pooled-QC grouping tightness and presence/absence of drift-correlated scores

## How to apply

Load the curated (post data-curation) feature data matrix for the candidate RM 8231 and SRM 1950 plasma samples from the MetaboLights deposit MTBLS1919 using pandas via TidyMS data-container conventions. Fit a PCA model with the scikit-learn implementation on the cleaned data matrix, then project the pooled/intrastudy QC injections and study samples into the PCA score space to obtain sample scores. Use those scores to (1) identify potential outliers, (2) reveal expected sample clusters such as the hypertriglyceridemic, diabetic, and African-American plasma pools of candidate RM 8231, and (3) track time-related systematic variance or signal drift across the run order. Produce PCA score plots mirroring the paper's Figure 4 (raw vs. curated data) and Figure S3 (before vs. after data curation) comparison, and write the sample-by-component scores to a CSV table. The decision point is comparing the curated model against the raw-data model: tighter pooled-QC grouping and absence of drift-correlated scores after curation indicate the upstream filters (Rt filter at 90 s, blank-based filter at 10-fold peak area, RSD filter at 20%, 100% intraclass prevalence filter with threshold area value of 5, and D-ratio filter at 10%) worked as intended.

## Related tools

- **scikit-learn** (Implements the PCA model whose score space is used to track data quality, outliers, clusters, and signal drift)
- **TidyMS** (Provides the curated feature data matrix (post data-curation) and data-container conventions used to load and organize samples and QC injections) — https://github.com/griquelme/tidyms
- **Pandas** (Loads and manipulates the extracted feature data matrix from the MetaboLights MTBLS1919 deposit before PCA fitting)
- **Python** (Programming language in which the PCA-based QC assessment pipeline is executed)

## Examples

```
from sklearn.decomposition import PCA; import pandas as pd; X = pd.read_csv('curated_feature_matrix.csv', index_col=0); scores = PCA(n_components=2).fit_transform(X); pd.DataFrame(scores).to_csv('pca_scores.csv')
```

## Evaluation signals

- PCA score plots must reproduce the qualitative sample clustering reported for the pooled QC models — curated data show tighter pooled-QC grouping without drift-correlated scores, matching Figure S3 of the paper.
- The sample-by-component scores table (pca_scores.csv) must contain one row per injection (pooled/intrastudy QC and study samples) and one column per retained principal component, with scores projecting from the curated feature matrix.
- Raw-data PCA model (comparison baseline) must show looser pooled-QC grouping and visible time-related systematic variance; the curated model must show the improvement.
- Sample clusters corresponding to the hypertriglyceridemic, diabetic, and African-American plasma pools of candidate RM 8231 must be discernible in the curated score plot.
- No outliers should appear only in the curated model that were not already present in the raw-data model — new outliers after curation would signal an over-aggressive filter step.

## Limitations

- PCA score plots only reveal drift that correlates with principal components; drift orthogonal to the top components may be missed.
- TidyMS, as described here, lacks feature annotation capability, so PCA scores cannot be linked to specific metabolite identities within this skill.
- No software version numbers are stated for TidyMS, Python, or scikit-learn, so exact numerical reproducibility across environments is not guaranteed.

## Evidence

- [results] Each step of the data curation process can be visually inspected by means of PCA models, as exemplified in Figure 4 with score plots displayed for the raw and curated data sets.: [evidence span withheld — anchor status: not_found]
- [discussion] The implementation in Python allows easy integration of results into state-of-the-art tools for model generation, such as scikit-learn: "The implementation in Python allows easy integration of results into state-of-the-art tools for model generation, such as scikit-learn"
- [methods] candidate RM 8231 Frozen Human Plasma Suite for Metabolomics (hypertriglyceridemic, diabetic, and African-American plasma samples): "candidate RM 8231 Frozen Human Plasma Suite for Metabolomics (hypertriglyceridemic, diabetic, and African-American plasma samples)"
- [methods] The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919.: "The mass spectrometry data have been deposited to the Metabo Lights public repository [38] with the data set identifier MTBLS1919."
- [results] all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated.: "all features with an RSD larger than 20% in pooled /intrastudy QC sample injections were eliminated."
