---
name: retention-time-regression-modeling
description: Use when authentic standards are unavailable and LC-HRMS annotations
  (e.g. CMDN-annotated metabolites with accurate m/z and RT) need chromatographic
  plausibility checks by predicting RT from SMILES/InChIKey structures.
when_to_use_negative:
- No compounds with experimentally measured LC-HRMS retention times are available
  to train or benchmark the regression models
- The validation goal is structural confirmation (MS2 spectral matching or authentic
  standard verification) rather than chromatographic plausibility of an annotation
- The chemical space of candidate metabolites falls far outside the pesticide library
  used for training, where RT regression would extrapolate unreliably
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3518
tools:
- name: XGBoost
  role: Best-performing RT regression model on the pesticide library (RMSE 0.89 min,
    R2 0.79, MAE 0.60 min; 95.1% of errors within 1 min)
- name: scikit-learn Random Forest
  role: Comparison RT regressor trained on the same descriptor matrix and 80:20 stratified
    split
- name: BRNN (Bayesian regularized neural network)
  role: Third comparison RT regressor benchmarked against XGBoost and Random Forest
- name: RDKit
  role: Computes molecular descriptors from SMILES/InChIKey structures to build the
    RT regression feature matrix
- name: CMDN
  role: Produces the annotated xenobiotic metabolite candidates (accurate m/z, RT,
    MS2) whose retention times are validated by the RT model
  repo: https://github.com/LeaveMeNotTonight/CMDN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:090295d1db93c2fb0609d9ae618db898a50a11ccadc9c243c7df21eed45cdb28
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - BRNN (Bayesian regularized neural network)
  - CMDN
  - RDKit
  - XGBoost
  - scikit-learn Random Forest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# retention-time-regression-modeling

## Summary

Trains and benchmarked machine-learning regressors (XGBoost, Random Forest, BRNN) on molecular descriptors to predict LC-HRMS retention times, enabling chromatographic validation of CMDN-annotated pesticide metabolites when authentic standards are unavailable.

## When to use

Use when authentic standards are unavailable and LC-HRMS annotations (e.g. CMDN-annotated metabolites with accurate m/z and RT) need chromatographic plausibility checks by predicting RT from SMILES/InChIKey structures.

## When NOT to use

- No compounds with experimentally measured LC-HRMS retention times are available to train or benchmark the regression models
- The validation goal is structural confirmation (MS2 spectral matching or authentic standard verification) rather than chromatographic plausibility of an annotation
- The chemical space of candidate metabolites falls far outside the pesticide library used for training, where RT regression would extrapolate unreliably

## Inputs

- 1,021-pesticide library with SMILES/InChIKey structures and experimentally measured LC-HRMS retention times
- Molecular descriptor feature matrix computed from SMILES/InChIKey strings (e.g. RDKit descriptors)
- CMDN-annotated candidate metabolites (accurate m/z, RT, MS2) needing chromatographic validation

## Outputs

- Trained RT regression models (XGBoost, Random Forest, BRNN)
- Per-model test-split metrics (RMSE, R2, MAE) written to rt_prediction_metrics.csv
- Predicted RTs and XGBoost error-within-1-min fraction for the held-out split and for candidate metabolites

## How to apply

Load the 1,021-pesticide library with SMILES/InChIKey structures and experimentally measured LC-HRMS retention times, then compute molecular descriptors (e.g. RDKit descriptors) to build the feature matrix for RT regression. Partition the descriptor-RT data into an 80:20 train/test split stratified on the RT distribution. Train three regressors with fixed hyperparameters for reproducibility: XGBoost, scikit-learn Random Forest, and a Bayesian regularized neural network (BRNN). Predict RTs on the held-out 20% split and compute RMSE, R2, and MAE per model, plus the fraction of XGBoost errors within 1 min of the measured RT. The best model (XGBoost in this paper: RMSE = 0.89 min, R2 = 0.79, MAE = 0.60 min, 95.1% of errors within 1 min) is then used to score whether predicted vs. observed RT differences for candidate metabolites are small enough to support the annotation.

## Related tools

- **XGBoost** (Best-performing RT regression model on the pesticide library (RMSE 0.89 min, R2 0.79, MAE 0.60 min; 95.1% of errors within 1 min))
- **scikit-learn Random Forest** (Comparison RT regressor trained on the same descriptor matrix and 80:20 stratified split)
- **BRNN (Bayesian regularized neural network)** (Third comparison RT regressor benchmarked against XGBoost and Random Forest)
- **RDKit** (Computes molecular descriptors from SMILES/InChIKey structures to build the RT regression feature matrix)
- **CMDN** (Produces the annotated xenobiotic metabolite candidates (accurate m/z, RT, MS2) whose retention times are validated by the RT model) — https://github.com/LeaveMeNotTonight/CMDN

## Examples

```
import xgboost as xgb; m = xgb.XGBRegressor(); m.fit(X_desc_train, rt_train); rt_pred = m.predict(X_desc_test)
```

## Evaluation signals

- XGBoost test-split RMSE ≈ 0.89 min, R2 ≈ 0.79, and MAE ≈ 0.60 min on the 20% held-out pesticide-library split
- ≥ 95.1% of XGBoost prediction errors fall within 1 min of the measured LC-HRMS retention time
- All three models (XGB, RF, BRNN) are trained on the same descriptor matrix and identical stratified 80:20 split, with hyperparameters fixed for reproducibility
- Per-model RMSE/R2/MAE values are written to rt_prediction_metrics.csv for comparison against the paper's reported metrics

## Limitations

- RT prediction is only one channel of multichannel validation; it complements, and does not replace, standard verification, time-course logistic modeling, or murine studies
- Performance is anchored to the chemical space of the 1,021-pesticide training library and its 2,886 biotransformed derivatives; extrapolation to other xenobiotic classes is unverified
- All scripts depend on a specific R environment (tidyverse, CluMSID, OrgMassSpecR, MSMSsim, msentropy, etc.) and compatible tools (MS-Dial ver. 4.80, BioTransformer 3.0), so the annotation pipeline feeding RT validation must be reproduced in that environment

## Evidence
