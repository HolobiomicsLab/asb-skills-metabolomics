---
name: lipid-ccs-prediction-model-training
description: Use when a curated lipid CCS training table (e.g., the 6394-measurement
  LiPydomics dataset) with one-hot lipid class, fatty acid modifier, and adduct features
  plus sum composition and m/z must be used to fit and evaluate an SVR CCS predictor.
when_to_use_negative:
- The input is already a trained model or a serialized prediction artifact — this
  skill only covers fitting and evaluating the SVR from the encoded measurement table.
- The task is HILIC retention time prediction rather than CCS prediction — the paper
  trains those models separately (23 lipid classes for RT vs 22 for CCS).
- The lipid class of interest is not among the 22 lipid classes covered by the paper's
  trained CCS prediction models, so no published reference metrics exist to validate
  against.
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Source of the curated CCS training dataset (6394 measurements) and package
    resources the model is built from
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Provides the support vector regression implementation with RBF kernel used
    to fit the CCS predictor on the 80% training split
- name: SciPy
  role: Scientific computing dependency supporting the Python data-handling steps
    of the training workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:bb47838f69e1f6a0d8cddde7cb517f213a46d7d0709d29355e9bb1f0e84c70e4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# lipid-ccs-prediction-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a support vector regression model with an RBF kernel that predicts lipid collision cross sections (CCS, in Å²) from 39-feature lipid encodings, replicating the LiPydomics CCS prediction approach. The skill exists to reproduce the published model's error profile and serialize it for downstream CCS prediction in HILIC-IM-MS lipidomics.

## When to use

Use when a curated lipid CCS training table (e.g., the 6394-measurement LiPydomics dataset) with one-hot lipid class, fatty acid modifier, and adduct features plus sum composition and m/z must be used to fit and evaluate an SVR CCS predictor.

## When NOT to use

- The input is already a trained model or a serialized prediction artifact — this skill only covers fitting and evaluating the SVR from the encoded measurement table.
- The task is HILIC retention time prediction rather than CCS prediction — the paper trains those models separately (23 lipid classes for RT vs 22 for CCS).
- The lipid class of interest is not among the 22 lipid classes covered by the paper's trained CCS prediction models, so no published reference metrics exist to validate against.

## Inputs

- Curated lipid CCS training dataset of 6394 measurements from the LiPydomics package resources
- 39-feature encoding per measurement: one-hot lipid class, fatty acid modifier, and adduct features plus sum composition and m/z

## Outputs

- Trained RBF-kernel SVR CCS prediction model serialized with joblib/pickle
- Metrics table (CSV) with MAE, MDAE, RMSE, and MDRE rows for the train (80%) and test (20%) splits

## How to apply

Load the curated CCS training dataset of 6394 measurements from the LiPydomics package resources and encode each measurement into the 39-feature vector (one-hot encoded lipid class, fatty acid modifier, and adduct features plus sum composition and m/z). Split the encoded data 80/20 into training and test sets following the documented protocol. Fit a support vector regression model with an RBF kernel on the 80% training subset using scikit-learn, then predict CCS values on both the training subset and the held-out 20% test subset and compute MAE, MDAE, RMSE, and median relative error (MDRE) for each. Serialize the trained model with joblib/pickle and write the metrics to a CSV table, accepting the run if the metrics match the reference values (train MAE/MDAE/RMSE = 1.05/0.55/1.79 Å²; test = 1.34/0.78/3.03 Å²; MDRE = 0.20%/0.27%) within reasonable tolerance.

## Related tools

- **LiPydomics** (Source of the curated CCS training dataset (6394 measurements) and package resources the model is built from) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Provides the support vector regression implementation with RBF kernel used to fit the CCS predictor on the 80% training split)
- **SciPy** (Scientific computing dependency supporting the Python data-handling steps of the training workflow)

## Examples

```
from sklearn.svm import SVR; model = SVR(kernel='rbf').fit(X_train, y_train); preds = model.predict(X_test)
```

## Evaluation signals

- Metrics table contains MAE, MDAE, RMSE, and MDRE rows for both train and test splits
- Training metrics reproduce MAE 1.05, MDAE 0.55, RMSE 1.79 Å² within reasonable tolerance
- Test metrics reproduce MAE 1.34, MDAE 0.78, RMSE 3.03 Å² within reasonable tolerance
- Median relative error (MDRE) is on the order of 0.20% (train) / 0.27% (test)
- Serialized model file loads with joblib/pickle and predicts CCS values for both the training subset and the held-out 20% test subset

## Limitations

- The models depend on the assembled experimental database of m/z and CCS of 45 lipid classes; the paper notes previous databases 'mostly cover mammalian lipid species, have limited coverage of bacterial lipids', so coverage gaps persist for bacterial lipids.
- Only 22 lipid classes had CCS prediction models trained, so CCS prediction outside those classes is unsupported by the published reference metrics.
- The package (including the lipid database and prediction models) is built to be extensible 'allowing easy expansion as more data becomes available', meaning published metrics reflect only the data available at training time.

## Evidence
