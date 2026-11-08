---
name: lipid-feature-one-hot-encoding
description: Use when training or applying lipid CCS prediction models that need tabular
  lipid measurements (lipid class, fatty acid sum composition, adduct, m/z) converted
  to a fixed-width numeric feature matrix for scikit-learn estimators.
when_to_use_negative:
- Input is already an encoded numeric feature matrix or a generic descriptor table
  (e.g., DeepCCS-style descriptors) — re-encoding would break compatibility with the
  LiPydomics-trained models
- The target lipid class or adduct is outside the 22 CCS prediction classes trained
  by LiPydomics; encoding an unseen category yields an uninformative vector
- The goal is lipid identification or stats/plotting analysis of HILIC-IM-MS data
  rather than CCS prediction — use the identification or stats modules instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LiPydomics
  role: provides the curated CCS training dataset of 6394 measurements and the lipid
    class/adduct vocabulary used to build the one-hot feature vectors
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: supplies the encoders for one-hot feature construction and the RBF-kernel
    SVR estimator fit on the encoded features
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:4e0dd37b41081ec152321f0631cb80b1090873398367809b0a7564b3115170ca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - SciPy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# lipid-feature-one-hot-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes lipid m/z and CCS measurements into fixed-width numeric feature vectors — one-hot encoded lipid class, fatty acid modifier, and adduct features plus sum composition and m/z — so machine learning models (e.g., scikit-learn SVR) can predict collision cross sections from simple but specialized lipid descriptors.

## When to use

Use when training or applying lipid CCS prediction models that need tabular lipid measurements (lipid class, fatty acid sum composition, adduct, m/z) converted to a fixed-width numeric feature matrix for scikit-learn estimators.

## When NOT to use

- Input is already an encoded numeric feature matrix or a generic descriptor table (e.g., DeepCCS-style descriptors) — re-encoding would break compatibility with the LiPydomics-trained models
- The target lipid class or adduct is outside the 22 CCS prediction classes trained by LiPydomics; encoding an unseen category yields an uninformative vector
- The goal is lipid identification or stats/plotting analysis of HILIC-IM-MS data rather than CCS prediction — use the identification or stats modules instead

## Inputs

- Curated lipid CCS measurement table (6394 records) with lipid class, fatty acid modifier, adduct, sum composition, m/z, and experimental CCS columns (LiPydomics package resources)
- Lipid identifiers for which CCS predictions are needed (class, sum composition, adduct, m/z)

## Outputs

- 39-column numeric feature matrix (one-hot lipid class, fatty acid modifier, and adduct blocks plus sum composition and m/z columns)
- Fitted encoder + SVR model serialized with joblib/pickle for reuse

## How to apply

Load the curated CCS measurement table (6394 experimental measurements shipped with the LiPydomics package) and encode each measurement into a 39-feature vector: one-hot encoded lipid class, fatty acid modifier, and adduct columns, augmented with the fatty acid sum composition and the precursor m/z. This 'simple but specialized feature set' is what LiPydomics used to train its CCS prediction models, so any reproduced model must use the same encoding to be comparable. Split the encoded matrix 80/20 into training and test subsets, fit an RBF-kernel SVR on the training subset, and serialize the fitted encoder+model pair (e.g., with joblib) so unseen lipid entries can be encoded identically at prediction time. Decision point: if a new lipid falls outside the one-hot class/adduct vocabulary learned during encoding, it cannot be represented and should be flagged rather than silently zero-encoded.

## Related tools

- **LiPydomics** (provides the curated CCS training dataset of 6394 measurements and the lipid class/adduct vocabulary used to build the one-hot feature vectors) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (supplies the encoders for one-hot feature construction and the RBF-kernel SVR estimator fit on the encoded features)

## Examples

```
from sklearn.preprocessing import OneHotEncoder; from sklearn.svm import SVR; X_enc = OneHotEncoder().fit_transform(X_cat); svr = SVR(kernel='rbf').fit(X_train, y_train); ccs_pred = svr.predict(X_test)
```

## Evaluation signals

- Encoded feature matrix has exactly 39 columns matching the documented encoding (one-hot lipid class, fatty acid modifier, adduct + sum composition and m/z)
- SVR metrics on the 80/20 split match the reference values within tolerance: train MAE/MDAE/RMSE ≈ 1.05/0.55/1.79 Å²; test ≈ 1.34/0.78/3.03 Å²
- Median relative error (MDRE) is ≈ 0.20% on training and ≈ 0.27% on test predictions
- Re-encoding an unseen lipid through the serialized encoder produces a valid 39-length vector and the SVR returns a CCS prediction
- Test RMSE (3.03 Å²) exceeding train RMSE (1.79 Å²) is expected and should not be treated as an encoding bug

## Limitations

- CCS prediction models were trained for only 22 lipid classes (and retention time models for 23), so lipids outside these classes cannot be encoded or predicted reliably
- Feature encoding covers class/adduct/sum-composition descriptors only — no 3D structure or IMS-specific physics — so accuracy is bounded by the 6394 experimental measurements
- Test-set error (RMSE 3.03 Å²) is notably higher than training error (1.79 Å²), indicating the model generalizes imperfectly outside the training distribution

## Evidence
