---
name: regression-error-metrics-evaluation
description: Use when a trained regression model (e.g., CCS or HILIC RT prediction
  from one-hot lipid class/fatty acid/adduct features) must be scored and its error
  metrics verified against published MAE/MDAE/RMSE/MDRE values.
when_to_use_negative:
- The task is classification, clustering, or unsupervised dimensionality analysis
  of lipidomics data rather than fitting a regression target (CCS/RT) — MAE/MDAE/RMSE/MDRE
  do not apply
- The lipid class of interest is absent from the trained models, which cover only
  22 CCS and 23 HILIC retention time classes out of the 45-class experimental database
- Only qualitative identification confidence levels are needed (lipid class/subclass
  and sum composition), not quantitative prediction accuracy
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LiPydomics
  role: Provides the curated CCS training dataset (6394 measurements) and the 596-lipid
    HILIC retention time subset via its data module, plus lipid feature encoding
  repo: https://github.com/dylanhross/lipydomics
- name: scikit-learn
  role: Supplies the RBF-kernel support vector regression and multivariate linear
    regression models and the error metric functions (MAE, MDAE, RMSE)
- name: SciPy
  role: Numerical support for computing median relative error and other statistics
    on predicted vs. observed values
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:cdf5d2a62adbd5e1eab7d1a5ad0df9834a36b608e31f1b2b49364dc75e366af4
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
yes_rate: 1.0
n_claims: 16
---

# regression-error-metrics-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A procedure for evaluating lipid-property regression models (SVR CCS predictors, multivariate linear HILIC retention-time predictors) by scoring held-out and training splits with MAE, MDAE, RMSE, and median relative error (MDRE), then comparing against reported reference values within tolerance.

## When to use

Use when a trained regression model (e.g., CCS or HILIC RT prediction from one-hot lipid class/fatty acid/adduct features) must be scored and its error metrics verified against published MAE/MDAE/RMSE/MDRE values.

## When NOT to use

- The task is classification, clustering, or unsupervised dimensionality analysis of lipidomics data rather than fitting a regression target (CCS/RT) — MAE/MDAE/RMSE/MDRE do not apply
- The lipid class of interest is absent from the trained models, which cover only 22 CCS and 23 HILIC retention time classes out of the 45-class experimental database
- Only qualitative identification confidence levels are needed (lipid class/subclass and sum composition), not quantitative prediction accuracy

## Inputs

- Encoded feature matrix (e.g., 39-feature one-hot lipid class/fatty acid modifier/adduct + sum composition + m/z vectors for 6394 CCS measurements)
- 26-feature encoded 596-lipid HILIC retention time training subset from the lipydomics data module
- Ground-truth CCS values (Å²) or HILIC retention times (min) paired with each feature row

## Outputs

- CSV metrics table with MAE, MDAE, RMSE (and MDRE) rows for train and test splits
- Serialized fitted model (pickle/joblib)
- Comparison of computed metrics against reference values (CCS: 1.05/0.55/1.79 train Å², 1.34/0.78/3.03 test Å², 0.20%/0.27% MDRE; RT: 0.11/0.08/0.15 min)

## How to apply

Split the encoded feature matrix (e.g., 39-feature vectors of 6394 CCS measurements, or 26-feature encodings of the 596-lipid HILIC RT subset) into train/test partitions following the documented protocol (SVR uses an 80/20 split). Fit the regression (SVR with RBF kernel via scikit-learn for CCS; multivariate linear regression for HILIC RT), predict on both the training subset and the held-out test subset, and compute MAE, MDAE, RMSE (and MDRE for CCS). Benchmark the computed values against the paper's references — CCS: train MAE/MDAE/RMSE = 1.05/0.55/1.79 Å², test = 1.34/0.78/3.03 Å², MDRE = 0.20%/0.27%; HILIC RT: test MAE/MDAE/RMSE = 0.11/0.08/0.15 min — accepting results only within a reasonable tolerance. Serialize the fitted model (pickle/joblib) and write a metrics table with MAE, MDAE, RMSE, and MDRE rows for each split to CSV.

## Related tools

- **LiPydomics** (Provides the curated CCS training dataset (6394 measurements) and the 596-lipid HILIC retention time subset via its data module, plus lipid feature encoding) — https://github.com/dylanhross/lipydomics
- **scikit-learn** (Supplies the RBF-kernel support vector regression and multivariate linear regression models and the error metric functions (MAE, MDAE, RMSE))
- **SciPy** (Numerical support for computing median relative error and other statistics on predicted vs. observed values)

## Examples

```
from sklearn.metrics import mean_absolute_error, median_absolute_error, mean_squared_error; mae = mean_absolute_error(y_test, y_pred); mdae = median_absolute_error(y_test, y_pred); rmse = mean_squared_error(y_test, y_pred)**0.5
```

## Evaluation signals

- Metrics CSV contains MAE, MDAE, RMSE (and MDRE) rows for both train and test splits
- SVR test metrics fall within tolerance of MAE 1.34, MDAE 0.78, RMSE 3.03 Å²; train metrics near 1.05/0.55/1.79 Å²; MDRE ≈ 0.20% (train) and 0.27% (test)
- HILIC RT model test MAE/MDAE/RMSE ≈ 0.11/0.08/0.15 min on the held-out subset
- Train/test split follows the documented 80/20 protocol for the 6394-measurement CCS set; RT uses the documented 596-lipid training/test partition
- Serialized model reloads and reproduces the recorded predictions/metrics

## Limitations

- Trained models cover only 22 (CCS) and 23 (RT) lipid classes of the 45 experimental classes, so metrics do not transfer to uncovered classes
- Reported metrics are split-dependent; reproducing them requires following the documented train/test partition exactly
- The 80/20 split protocol is specific to the SVR CCS task; the HILIC RT task uses the documented 596-lipid subset partition

## Evidence

- [results] The SVR model was trained using 80/20 split, achieving MAE 1.05, MDAE 0.55, RMSE 1.79 Å² on training data, and MAE 1.34, MDAE 0.78, RMSE 3.03 Å² on test data.: [evidence span withheld — anchor status: not_found]
- [results] A multivariate linear regression model trained on HILIC retention time data achieved MAE, MDAE, and RMSE of 0.11, 0.08, and 0.15 min on the test set.: [evidence span withheld — anchor status: not_found]
