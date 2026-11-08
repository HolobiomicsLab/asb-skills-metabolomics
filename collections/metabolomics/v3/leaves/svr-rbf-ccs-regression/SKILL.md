---
name: svr-rbf-ccs-regression
description: Use when a lipid CCS value must be predicted for a lipid class/adduct
  covered by the LiPydomics experimental CCS database, or when reproducing/validating
  the LiPydomics SVR CCS prediction performance against reported MAE/MDAE/RMSE/MDRE
  metrics.
when_to_use_negative:
- Predicting HILIC retention times rather than CCS — the package trains RT models
  for 23 lipid classes and CCS models for 22, so RT questions need the retention-time
  predictors instead
- The target lipid class or adduct is outside the 45 lipid classes covered by the
  assembled experimental m/z and CCS database, so no trained class-specific SVR exists
- The input is already a feature table with model outputs — the encoding step (one-hot
  lipid class/adduct plus sum composition and m/z) is already done and only evaluation
  is needed
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3474
tools:
- name: LiPydomics
  role: Supplies the curated experimental CCS training dataset (6394 measurements)
    and the lipid class/adduct feature encoding used as SVR input
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Fits the support vector regression model with an RBF kernel on the 80% training
    subset and produces CCS predictions
- name: SciPy
  role: Supports numerical computation in the Python CCS prediction workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:2f65e29f07345147543d14af1abdd2f68d527e6d072e824c81160fdce8dd19f7
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
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# svr-rbf-ccs-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a support vector regression model with an RBF kernel (scikit-learn) to predict lipid collision cross sections (CCS) from LiPydomics' curated 6394-measurement CCS database, using a 39-feature encoding (one-hot lipid class, fatty acid modifier, and adduct, plus sum composition and m/z), and validates against reported error metrics.

## When to use

Use when a lipid CCS value must be predicted for a lipid class/adduct covered by the LiPydomics experimental CCS database, or when reproducing/validating the LiPydomics SVR CCS prediction performance against reported MAE/MDAE/RMSE/MDRE metrics.

## When NOT to use

- Predicting HILIC retention times rather than CCS — the package trains RT models for 23 lipid classes and CCS models for 22, so RT questions need the retention-time predictors instead
- The target lipid class or adduct is outside the 45 lipid classes covered by the assembled experimental m/z and CCS database, so no trained class-specific SVR exists
- The input is already a feature table with model outputs — the encoding step (one-hot lipid class/adduct plus sum composition and m/z) is already done and only evaluation is needed

## Inputs

- Curated lipid CCS training dataset of 6394 measurements from the LiPydomics package resources
- Per-lipid descriptor inputs: lipid class, fatty acid modifier, adduct, sum composition, and m/z

## Outputs

- Serialized SVR model (joblib/pickle)
- CSV metrics table with MAE, MDAE, RMSE, and MDRE rows for the train and test splits
- Predicted CCS values (Å²) for each encoded lipid measurement in train and test subsets

## How to apply

Load the curated CCS training dataset of 6394 measurements shipped with the LiPydomics package resources and encode each measurement into the 39-feature vector: one-hot encoded lipid class, fatty acid modifier, and adduct features plus sum composition and m/z. Split the encoded data 80/20 into training and test subsets following the documented protocol. Fit a support vector regression model with an RBF kernel on the 80% training subset using scikit-learn, then predict CCS on both subsets and compute MAE, MDAE, RMSE, and median relative error (MDRE). Serialize the fitted SVR with joblib/pickle and write the metrics table to CSV. Accept the result if train MAE/MDAE/RMSE ≈ 1.05/0.55/1.79 Å² and test MAE/MDAE/RMSE ≈ 1.34/0.78/3.03 Å², with MDRE ≈ 0.20% (train) and 0.27% (test), within a reasonable tolerance.

## Related tools

- **LiPydomics** (Supplies the curated experimental CCS training dataset (6394 measurements) and the lipid class/adduct feature encoding used as SVR input) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Fits the support vector regression model with an RBF kernel on the 80% training subset and produces CCS predictions)
- **SciPy** (Supports numerical computation in the Python CCS prediction workflow)

## Examples

```
from sklearn.svm import SVR; svr = SVR(kernel='rbf'); svr.fit(X_train, y_train); preds = svr.predict(X_test)
```

## Evaluation signals

- Train-split metrics reproduce MAE 1.05, MDAE 0.55, RMSE 1.79 Å² (reference values) within tolerance
- Test-split metrics reproduce MAE 1.34, MDAE 0.78, RMSE 3.03 Å² and MDRE ≈ 0.27% within tolerance
- Encoded feature matrix has exactly 39 columns (one-hot lipid class + fatty acid modifier + adduct, plus sum composition and m/z) and 6394 rows before the 80/20 split
- Serialized model file loads and reproduces the recorded test predictions without retraining
- Metrics CSV contains MAE, MDAE, RMSE, and MDRE rows for both train and test splits

## Limitations

- Trained CCS prediction models exist for 22 lipid classes only, so CCS prediction is unavailable for lipid classes outside the trained set
- Experimental CCS collections do not comprehensively cover the vast lipid chemical space, so predictions fill gaps but rely on the class/composition coverage of the assembled database
- Reported MDRE values (0.20% train / 0.27% test) imply high precision only within the represented lipid classes and adducts; extrapolation outside is unsupported

## Evidence
