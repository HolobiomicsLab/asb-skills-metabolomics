---
name: train-test-split-metrics-evaluation
description: Use when a trained CCS prediction model must be validated against published
  error metrics (MAE, MDAE, RMSE, MDRE), or when a held-out train/test split evaluation
  protocol must be reproduced for lipid CCS regression models.
when_to_use_negative:
- Prediction for lipid classes outside the 22 classes for which CCS models were trained
  — extend the database instead, since the package is built to be 'highly extensible
  and customizable, allowing easy expansion as more data becomes available'
- The task is HILIC retention time prediction rather than CCS prediction — retention
  time models were trained for a different set of 23 lipid classes
- A CCS value for a covered lipid class/adduct is already available in the predicted
  lipid database (145,388 entries) — lookup, not regression evaluation, is appropriate
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LiPydomics
  role: Supplies the curated CCS training dataset and the package resources containing
    the trained SVR prediction models being evaluated
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Fits the support vector regression model with RBF kernel and provides train/test
    splitting; model serialization uses joblib/pickle
- name: SciPy
  role: Supporting scientific computing library used alongside scikit-learn in the
    Python evaluation workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:8c22718e142a33354b3178981f2c19162bfd9e82efbf26e7a01058ceb5dcadfa
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# train-test-split-metrics-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the LiPydomics CCS support vector regression (SVR) evaluation: an 80/20 train/test split of the curated 6394-measurement lipid CCS dataset, RBF-kernel SVR fitting, and computation of MAE, MDAE, RMSE, and median relative error (MDRE) to verify the published error metrics.

## When to use

Use when a trained CCS prediction model must be validated against published error metrics (MAE, MDAE, RMSE, MDRE), or when a held-out train/test split evaluation protocol must be reproduced for lipid CCS regression models.

## When NOT to use

- Prediction for lipid classes outside the 22 classes for which CCS models were trained — extend the database instead, since the package is built to be 'highly extensible and customizable, allowing easy expansion as more data becomes available'
- The task is HILIC retention time prediction rather than CCS prediction — retention time models were trained for a different set of 23 lipid classes
- A CCS value for a covered lipid class/adduct is already available in the predicted lipid database (145,388 entries) — lookup, not regression evaluation, is appropriate

## Inputs

- Curated lipid CCS training dataset of 6394 measurements from LiPydomics package resources (45 lipid classes experimental CCS database)
- Per-measurement 39-feature vector: one-hot encoded lipid class, fatty acid modifier, and adduct features plus sum composition and m/z
- 80/20 train/test split designation (training subset for fitting, held-out 20% for evaluation)

## Outputs

- Serialized trained RBF-kernel SVR model (joblib/pickle)
- CSV metrics table with MAE, MDAE, RMSE, and MDRE rows for both train and test splits
- Comparison against reference values: train MAE/MDAE/RMSE = 1.05/0.55/1.79 Å²; test = 1.34/0.78/3.03 Å²; MDRE = 0.20%/0.27%

## How to apply

Load the curated CCS training dataset of 6394 measurements from the LiPydomics package resources and encode each measurement into the 39-feature vector (one-hot encoded lipid class, fatty acid modifier, and adduct features plus sum composition and m/z). Split the encoded data 80/20 into training and test subsets following the documented protocol. Fit a support vector regression model with an RBF kernel (scikit-learn) on the 80% training subset, then predict CCS values on both the training subset and the held-out 20% test subset, computing MAE, MDAE, RMSE, and MDRE for each split. Serialize the fitted SVR with joblib/pickle and write the metrics to a CSV table. The rationale for the four metrics is that MAE/RMSE capture absolute error in Å² while MDAE and MDRE capture median-absolute and median relative error, which is how the authors benchmark against lipid-specific CCS predictors such as LipidCCS and DeepCCS.

## Related tools

- **LiPydomics** (Supplies the curated CCS training dataset and the package resources containing the trained SVR prediction models being evaluated) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Fits the support vector regression model with RBF kernel and provides train/test splitting; model serialization uses joblib/pickle)
- **SciPy** (Supporting scientific computing library used alongside scikit-learn in the Python evaluation workflow)

## Examples

```
from sklearn.svm import SVR; from sklearn.model_selection import train_test_split; X_tr, X_te, y_tr, y_te = train_test_split(X39, y_ccs, test_size=0.2); model = SVR(kernel='rbf').fit(X_tr, y_tr)
```

## Evaluation signals

- Metrics table contains MAE, MDAE, RMSE, and MDRE rows for both train and test splits
- Computed train metrics match MAE/MDAE/RMSE = 1.05/0.55/1.79 Å² and test metrics match 1.34/0.78/3.03 Å² within a reasonable tolerance
- MDRE values fall near the reported 0.20% (train) and 0.27% (test)
- Encoded feature matrix has exactly 39 columns per measurement and 6394 rows total before the 80/20 split
- Serialized SVR model deserializes with joblib/pickle and reproduces the test-set predictions

## Limitations

- Performance claims hold only for the 22 lipid classes with trained CCS models; coverage of the broader lipid chemical space is incomplete since 'these collections do not yet comprehensively cover the vast lipid chemical space'
- Evaluation uses a single 80/20 split, so metrics may vary with a different random split or cross-validation scheme
- Reference metrics were obtained with the specific feature encoding (39 features including one-hot class/adduct and m/z); deviating from this encoding invalidates the comparison

## Evidence
