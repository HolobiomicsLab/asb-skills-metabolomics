---
name: regression-model-validation-metrics
description: Use when scoring an SVR or similar regression model's predictive accuracy
  with MRE, RMSE and R2 under cross-validation and test-set validation, and checking
  per-adduct median relative errors against a ≤2% target.
when_to_use_negative:
- The task is classification or annotation ranking rather than predicting a continuous
  quantity — median relative error and RMSE do not apply
- Predictions lack matched reference CCS values (e.g. no measured CCS in the McLean
  Compendium for the ion), so MRE/RMSE/R2 cannot be computed
- Input adducts are outside the trained scope, such as dimers ([2M+H]+, [2M-H]-),
  other less common adducts or in-source fragments
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: CCSP 2.0
  role: Jupyter notebook whose SVR models are trained and whose predictions are evaluated
    with MRE, RMSE and R2 metrics
  repo: https://github.com/facundof2016/CCSP2.0
- name: SciKit-Learn
  role: Provides SVR, GridSearchCV (5-fold CV minimizing RMSECV) and RFECV (stop when
    RMSECV increases) used to build and validate the models; requires V1.0.2 or later
- name: Mordred
  role: Supplies the 1613 2D molecular descriptors whose filtered/RFE-selected subsets
    feed the evaluated models
- name: McLean CCS Compendium
  role: Reference dataset of drift-tube CCS values used as ground truth for computing
    median relative errors and RMSEV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:ae2f5ad108b5ad01b42a9dbe58139c77a269199ecabd54a590e507007d317da7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - McLean CCS Compendium
  - Mordred
  - Numpy
  - Pandas
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# regression-model-validation-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates regression models (here, linear-kernel SVR CCS predictors) using median relative error, RMSE and R2 across calibration, 5-fold cross-validation and a 30% external hold-out set, benchmarked against the McLean CCS Compendium.

## When to use

Use when scoring an SVR or similar regression model's predictive accuracy with MRE, RMSE and R2 under cross-validation and test-set validation, and checking per-adduct median relative errors against a ≤2% target.

## When NOT to use

- The task is classification or annotation ranking rather than predicting a continuous quantity — median relative error and RMSE do not apply
- Predictions lack matched reference CCS values (e.g. no measured CCS in the McLean Compendium for the ion), so MRE/RMSE/R2 cannot be computed
- Input adducts are outside the trained scope, such as dimers ([2M+H]+, [2M-H]-), other less common adducts or in-source fragments

## Inputs

- Cleaned McLean Compendium adduct datasets (567 [M-H]-, 518 [M+H]+ and 461 [M+Na]+ ions) with measured CCS values
- Z-transformed descriptor matrix (1613 Mordred 2D descriptors, filtered to RFE-selected subsets of ~370, ~303 and ~414 descriptors per adduct)
- Per-adduct trained linear-kernel SVR model files and their selected-descriptor lists
- CCS predictions paired with reference CCS values for held-out molecules

## Outputs

- Per-adduct MRE, RMSE and R2 for calibration, cross-validation and external validation conditions
- Median relative errors per adduct type compared to the ≤2% target (e.g. 1.25%, 1.73%, 1.87%)
- RMSEV values in Å2 for each adduct model
- GridSearchCV-selected (C, epsilon) hyperparameter pairs per evaluation round
- Benchmark comparison table across 101 randomly-selected test sets versus CCSbase, DeepCCS and AllCCS

## How to apply

After training, split performance reporting into three conditions: calibration fit, cross-validation, and external validation on a randomly assigned 30% hold-out set (70/30 split taken before Z-transforming descriptors with training-set mean and standard deviation). For linear-kernel SVR models, tune C (seven options from 0.00390625 to 0.25) and epsilon (0.5, 1, 5) via GridSearchCV with 5-fold cross-validation, choosing the pair that minimizes RMSECV; in feature elimination, stop RFECV removing the five lowest-ranked Mordred descriptors when RMSECV increases. Report MRE, RMSE and R2 per adduct type ([M-H]-, [M+H]+, [M+Na]+), using median relative error as the headline metric with a target of ≤2% on the hold-out set (paper reports 1.25%, 1.73% and 1.87%). For external comparison, re-evaluate over 101 randomly-selected test sets and benchmark against CCSbase, DeepCCS and AllCCS. Treat RMSE values in Å2 (paper RMSEV of 4.998, 5.127, 6.612 for the three adducts) as scale-sensitive checks complementing the relative-error metrics.

## Related tools

- **CCSP 2.0** (Jupyter notebook whose SVR models are trained and whose predictions are evaluated with MRE, RMSE and R2 metrics) — https://github.com/facundof2016/CCSP2.0
- **SciKit-Learn** (Provides SVR, GridSearchCV (5-fold CV minimizing RMSECV) and RFECV (stop when RMSECV increases) used to build and validate the models; requires V1.0.2 or later)
- **Mordred** (Supplies the 1613 2D molecular descriptors whose filtered/RFE-selected subsets feed the evaluated models)
- **McLean CCS Compendium** (Reference dataset of drift-tube CCS values used as ground truth for computing median relative errors and RMSEV)

## Examples

```
GridSearchCV(SVR(kernel='linear'), param_grid={'C':[0.00390625,0.0078125,0.015625,0.03125,0.0625,0.125,0.25],'epsilon':[0.5,1,5]}, cv=5)
```

## Evaluation signals

- Per-adduct median relative error on the 30% hold-out set is at or below ~2% (paper reports 1.25%, 1.73% and 1.87% for [M-H]-, [M+H]+ and [M+Na]+)
- RMSECV does not increase when descriptors are removed during RFECV; validation stopping corresponds to the minimum-RMSECV point
- Reported RMSEV falls in the Å2 range reported by the paper (4.998, 5.127 and 6.612 Å2 for the three adducts)
- R2, RMSE and MRE are reported for all three evaluation conditions (calibration, cross-validation, external validation), not just training fit
- Benchmark over 101 randomly-selected test sets shows CCSP 2.0 performing better than or equal to CCSbase, DeepCCS and AllCCS in most cases

## Limitations

- Training sets do not support dimers such as [2M+H]+ or [2M-H]-, or other less common adducts or in-source fragments found in ESI-MS experiments
- CCSP 2.0 requires scikit-learn V1.0.2 or later; package versions must match requirements.txt
- Predictions were assessed against drift-tube CCS values from the McLean Compendium; CCS prediction across multiple IM-MS instrument types (drift tube, travelling wave) was not yet explored
- Benchmark method names are not detailed in every section; the three compared ML methods are referenced only as 'three popular machine learning CCS prediction methods reported in the literature'

## Evidence

- [readme] scikit-learn version requirement for running the validated models: [evidence span withheld — anchor status: external_doc]
