---
name: random-forest-cross-validated-classification
description: Use when you need a supervised classification baseline on peak-aligned
  metabolomics feature tables (e.g., cosine-similarity or spectral-entropy similarity
  tables) with labeled bioactivity groups, reported as accuracy, precision, recall,
  F1, and ROC AUC.
when_to_use_negative:
- When the goal is the paper's proposed method itself — direct CNN prediction from
  raw LC-MS/MS encoded as 2D grayscale molecular fingerprint images — this baseline
  skill bypasses that image-encoding pipeline entirely
- When features require chemical identification or annotation before modeling; this
  skill deliberately uses raw feature/similarity tables without compound identification
- When the sample labels are continuous rather than the paper's discretized classes
  (normalized DPPH values <0.4 vs >0.6), since intermediate values (0.4–0.6) are excluded
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and extracts MS1/MS2 m/z pairs and intensities
    that feed the feature tables the RF is trained on
- name: pandas
  role: Merges equivalent features (average MS1, RT, MS/MS similarity > 0.8) into
    the two unified feature tables used as RF inputs
- name: scikit-learn
  role: Provides RandomForestClassifier (v1.3.0, default hyperparameters) with 5-fold
    cross-validation and metric computation (accuracy, precision, recall, F1, ROC
    AUC)
- name: numpy
  role: Numerical handling of intensity matrices and similarity-score computation
    supporting feature-table construction
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:94a4d90dde72c7a56ddb0d3741fde0db16f85b5048b97f4936f0e35b253797be
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MS-DIAL
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# random-forest-cross-validated-classification

## Summary

Trains Random Forest classifiers (scikit-learn 1.3.0, default hyperparameters, 5-fold cross-validation) on LC-MS/MS-derived feature tables to predict DPPH bioactivity class for 545 wild soybean accessions, establishing the conventional-ML baseline that the MS2toImg-CNN framework is benchmarked against.

## When to use

Use when you need a supervised classification baseline on peak-aligned metabolomics feature tables (e.g., cosine-similarity or spectral-entropy similarity tables) with labeled bioactivity groups, reported as accuracy, precision, recall, F1, and ROC AUC.

## When NOT to use

- When the goal is the paper's proposed method itself — direct CNN prediction from raw LC-MS/MS encoded as 2D grayscale molecular fingerprint images — this baseline skill bypasses that image-encoding pipeline entirely
- When features require chemical identification or annotation before modeling; this skill deliberately uses raw feature/similarity tables without compound identification
- When the sample labels are continuous rather than the paper's discretized classes (normalized DPPH values <0.4 vs >0.6), since intermediate values (0.4–0.6) are excluded

## Inputs

- MS-DIAL-aligned peak data (profile MS1, centroid MS/MS) from 545 wild soybean (Glycine soja Sieb. & Zucc.) extract LC-MS/MS runs
- Two similarity-based feature tables: one built with cosine-similarity MS/MS scores and one with spectral-entropy MS/MS scores
- Normalized DPPH radical scavenging values per accession (categorical labels: <0.4 low-activity, >0.6 high-activity)

## Outputs

- Per-fold and mean 5-fold cross-validation metrics: accuracy, precision, recall, F1-score
- RF ROC curve on the DPPH task with AUC ≈ 0.817 (paper Figure 4C)
- Trained Random Forest models per feature table (cosine similarity and spectral entropy)

## How to apply

First build unified feature tables from MS-DIAL-preprocessed peak data: keep peak intensities > 10^-4, apply TIC normalization with 1-minute RT alignment tolerance, compute pairwise similarity scores (exponential-decay for MS1 m/z and RT; cosine similarity and spectral entropy for MS/MS), and merge features when the average of the MS1, RT, and MS/MS similarity scores exceeds 0.8, otherwise add as a new entry. Label the 545 wild soybean accessions from normalized DPPH radical scavenging values, assigning normalized values below 0.4 to the low-activity group and above 0.6 to the high-activity group. Train a RandomForestClassifier (scikit-learn 1.3.0, default hyperparameters) on each feature table (cosine-similarity and spectral-entropy variants) with 5-fold cross-validation, and compute accuracy, precision, recall, and F1-score. Generate the ROC curve and verify the AUC is approximately 0.817, matching Figure 4C of the paper; these RF metrics serve as the baseline for comparison against the MS2toImg-CNN model.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and extracts MS1/MS2 m/z pairs and intensities that feed the feature tables the RF is trained on)
- **pandas** (Merges equivalent features (average MS1, RT, MS/MS similarity > 0.8) into the two unified feature tables used as RF inputs)
- **scikit-learn** (Provides RandomForestClassifier (v1.3.0, default hyperparameters) with 5-fold cross-validation and metric computation (accuracy, precision, recall, F1, ROC AUC))
- **numpy** (Numerical handling of intensity matrices and similarity-score computation supporting feature-table construction)

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.model_selection import cross_val_score; rf = RandomForestClassifier(); scores = cross_val_score(rf, X_features, y_dpph_labels, cv=5, scoring='roc_auc')
```

## Evaluation signals

- 5-fold cross-validation ROC AUC on the DPPH task is approximately 0.817, matching Figure 4C
- Accuracy, precision, recall, and F1 from 5-fold cross-validation match the paper's Figure 4C and Figure 5 reported values within reasonable tolerance
- Feature tables are built with the specified merge rule: features merge only when the average of MS1, RT, and MS/MS similarity scores exceeds 0.8
- Labels obey the paper's discretization: normalized DPPH values <0.4 → low-activity group, >0.6 → high-activity group; samples in between are excluded
- Only peak intensity values greater than 10^-4 are retained and TIC normalization with 1-minute RT alignment tolerance was applied before table construction

## Limitations

- Random Forest baseline was outperformed by the MS2toImg-CNN model, so it is a comparison baseline rather than the paper's best method
- Conventional feature-table pipelines are sensitive to alignment errors caused by low instrument reproducibility, since they rely on precise feature matching across samples
- Only accessions with normalized DPPH values below 0.4 or above 0.6 are labeled; intermediate-activity samples are discarded, reducing usable training data
- Default hyperparameters (scikit-learn 1.3.0) were used without tuning, so performance may not reflect an optimized RF

## Evidence

- [methods] RF training and CV setup: [evidence span withheld — anchor status: not_found]
- [methods] Expected AUC: [evidence span withheld — anchor status: not_found]
