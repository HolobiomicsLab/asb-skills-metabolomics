---
name: lcmsms-feature-table-construction
description: Use when MS-DIAL-aligned peak intensity data (profile MS1, centroid MS/MS)
  from many LC-MS/MS samples must be merged into feature tables via similarity-based
  equivalence rules for downstream ML bioactivity prediction.
when_to_use_negative:
- 'The goal is the MS2toImg image-based route: raw LC-MS/MS data are converted directly
  to grayscale molecular fingerprint images, bypassing explicit feature selection,
  alignment, or annotation'
- A curated, already-aligned feature matrix is available and only model training/benchmarking
  (PCA, OPLS-DA, RF, CNN) is required
- The task requires structural annotation of individual features (e.g., CANOPUS, MetDNA,
  Spec2Vec) rather than sample-level bioactivity prediction
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data and extracts MS1/MS2 m/z pairs with intensities;
    performs peak filtering (intensity > 10^-4), TIC normalization, adduct selection,
    and alignment with 1-minute RT tolerance
- name: pandas
  role: Merges equivalent features into the unified feature table when the average
    of MS1, RT, and MS/MS similarity scores exceeds 0.8
- name: scikit-learn
  role: Trains Random Forest classifiers (v1.3.0, default hyperparameters) on the
    constructed feature tables with 5-fold cross-validation to produce accuracy, precision,
    recall, F1, and ROC AUC
- name: numpy
  role: Numerical support for similarity computations (exponential-decay MS1/RT scores,
    cosine similarity, spectral entropy)
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:9f6252aca9b648325708c6aed63098f1dcf08309991db3b0e3754bca4838e24e
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lcmsms-feature-table-construction

## Summary

Constructs unified feature tables from raw untargeted LC-MS/MS peak data by merging MS-DIAL-detected features using composite MS1/RT/MS/MS similarity scores, producing labeled matrices suitable for Random Forest bioactivity baselines (e.g., DPPH classification of 545 wild soybean accessions).

## When to use

Use when MS-DIAL-aligned peak intensity data (profile MS1, centroid MS/MS) from many LC-MS/MS samples must be merged into feature tables via similarity-based equivalence rules for downstream ML bioactivity prediction.

## When NOT to use

- The goal is the MS2toImg image-based route: raw LC-MS/MS data are converted directly to grayscale molecular fingerprint images, bypassing explicit feature selection, alignment, or annotation
- A curated, already-aligned feature matrix is available and only model training/benchmarking (PCA, OPLS-DA, RF, CNN) is required
- The task requires structural annotation of individual features (e.g., CANOPUS, MetDNA, Spec2Vec) rather than sample-level bioactivity prediction

## Inputs

- MS-DIAL-preprocessed LC-MS/MS peak data (profile MS1 and centroid MS/MS precursor/fragment m/z pairs with intensities) from 545 wild soybean (Glycine soja Sieb. & Zucc.) extract accessions
- MS-DIAL alignment settings: adduct ions [M-H]-, [M-H2O-H]-, [M+Na-2H]-, [M-K-2H]-, [M-FA-H]-; RT alignment tolerance 1 minute
- Bioassay label source: normalized DPPH radical scavenging values per accession

## Outputs

- Two unified feature tables over the 545 accessions: one built with MS/MS cosine similarity, one with spectral entropy
- Per-feature-pair composite similarity scores (MS1 exponential-decay, RT, MS/MS) used for the merge decision
- Labeled binary activity classes (low < 0.4 vs high > 0.6 normalized DPPH)
- RF baseline metrics per table: accuracy, precision, recall, F1 from 5-fold cross-validation, and ROC AUC (≈0.817)

## How to apply

Load the MS-DIAL-aligned peak data and retain only peak intensity values greater than 10^-4, with TIC-normalized intensities and alignment performed at a 1-minute retention-time tolerance. For each pair of detected features, compute an exponential-decay similarity score for MS1 m/z (sigma set to the empirical standard deviation of observed MS1 values) and for retention time (sigma set to the search tolerance), plus MS/MS spectral similarity computed two ways: cosine similarity and spectral entropy. Merge features into a unified pandas table using the rule that when the average of the three similarity scores (MS1, RT, MS/MS) exceeds 0.8 the features are considered equivalent and merged, otherwise the feature is added as a new entry; build one table per MS/MS similarity metric (cosine similarity table and spectral entropy table). Label samples from the bioassay readout (e.g., normalized DPPH radical scavenging: <0.4 = low-activity, >0.6 = high-activity) and train Random Forest classifiers (scikit-learn 1.3.0, default hyperparameters) with 5-fold cross-validation. The rationale is that this alignment/merging step defines the feature space that conventional baselines (RF, PCA, OPLS-DA) depend on, in contrast to the paper's image-based MS2toImg route that bypasses it.

## Related tools

- **MS-DIAL** (Preprocesses raw LC-MS/MS data and extracts MS1/MS2 m/z pairs with intensities; performs peak filtering (intensity > 10^-4), TIC normalization, adduct selection, and alignment with 1-minute RT tolerance)
- **pandas** (Merges equivalent features into the unified feature table when the average of MS1, RT, and MS/MS similarity scores exceeds 0.8)
- **scikit-learn** (Trains Random Forest classifiers (v1.3.0, default hyperparameters) on the constructed feature tables with 5-fold cross-validation to produce accuracy, precision, recall, F1, and ROC AUC)
- **numpy** (Numerical support for similarity computations (exponential-decay MS1/RT scores, cosine similarity, spectral entropy))

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.model_selection import cross_val_score; scores = cross_val_score(RandomForestClassifier(), X_table, y_dpph, cv=5)
```

## Evaluation signals

- ROC AUC of the RF baseline on the DPPH task should be approximately 0.817 as reported for the spectral-data RF baseline (Figure 4C)
- Reproduced accuracy, precision, recall, and F1 from 5-fold cross-validation must match the paper's Figure 4C / Figure 5 results within reasonable tolerance
- Merge rule check: no two retained features in a table have an average MS1+RT+MS/MS similarity score above 0.8 (they should have been merged)
- All retained peak intensities exceed 10^-4 and are TIC-normalized; alignment respects the 1-minute RT tolerance
- Label assignment check: no sample with normalized DPPH value between 0.4 and 0.6 is assigned to the low- or high-activity group

## Limitations

- The 0.8 average-similarity merge threshold is a hard binary decision; the paper's image-based approach was motivated precisely because this feature-matching pipeline is sensitive to alignment errors from low instrument reproducibility
- Labeling discards the middle band: normalized DPPH values between 0.4 and 0.6 are assigned to neither the low- nor high-activity group, reducing usable samples
- Repository code is restricted: redistribution, modification, or use for other research or commercial purposes is not permitted until formal publication; RF results depend on scikit-learn 1.3.0 default hyperparameters

## Evidence
