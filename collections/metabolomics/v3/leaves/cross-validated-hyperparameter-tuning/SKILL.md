---
name: cross-validated-hyperparameter-tuning
description: Use when training retention-time prediction models on HILIC or RPLC libraries
  (e.g. Retip training sets with CDK descriptors) and you need tuned models plus an
  overfitting check comparing training vs test/validation errors.
when_to_use_negative:
- Compounds fall outside the chemical landscape of the training set (the paper warns
  results may be more erroneous), e.g. complex high-molecular-weight lipids such as
  triacylglycerols or phosphatidylcholines that were excluded from training
- No experimental retention times are available for the target chromatographic method,
  so there is no supervised target to tune against
- The goal is a narrow retention-time search window (e.g. discriminating closely eluting
  isomers in a 12 min UPLC run) — 1 min MAEs are inadequate for that purpose
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Retip
  role: R package supplying getCD(), proc.data(), RT.spell(), and the fit.* model-fitting
    functions used for tuning and prediction
  repo: https://github.com/olobion/Retip
- name: caret
  role: 80:20 data splitting (indexed on XlogP) and 10-fold cross-validation for the
    non-Keras models
- name: Keras
  role: Tuned via dense_unit, epochs, dropout with validation_split=0.2; best-performing
    model with MAE 0.78 min (HILIC) and 0.57 min (RPLC)
- name: XGBoost
  role: Grid search over nrounds, max_depth, eta with fixed gamma, colsample_bytree,
    subsample, min_child_weight
- name: LightGBM
  role: Optimal nrounds selected automatically via best iteration
- name: rcdk
  role: Computes the 286 CDK chemical descriptors that serve as tuning features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:6fe3edbfee145c41d54f9ef6a29bf439a9c772276c74ffbb602e1f669c31ed9d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - Keras
  - LightGBM
  - R
  - Retip
  - XGBoost
  - caret
  - rcdk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# cross-validated-hyperparameter-tuning

## Summary

Fits the five Retip machine learning models (random forest, BRNN, XGBoost, LightGBM, Keras) with per-algorithm hyperparameter tuning and cross-validation, then quantifies overfitting via MAE differences between training, test, and validation sets.

## When to use

Use when training retention-time prediction models on HILIC or RPLC libraries (e.g. Retip training sets with CDK descriptors) and you need tuned models plus an overfitting check comparing training vs test/validation errors.

## When NOT to use

- Compounds fall outside the chemical landscape of the training set (the paper warns results may be more erroneous), e.g. complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines that were excluded from training
- No experimental retention times are available for the target chromatographic method, so there is no supervised target to tune against
- The goal is a narrow retention-time search window (e.g. discriminating closely eluting isomers in a 12 min UPLC run) — 1 min MAEs are inadequate for that purpose

## Inputs

- HILIC training set: 880 compounds from MoNA with experimental retention times
- RPLC training set: 398 compounds from the RIKEN PlaSMA database with experimental retention times
- HILIC test set (143 compounds) and RPLC test set (96 compounds)
- External validation sets: 377 Pathogen Box compounds (HILIC) and 358 Ophiorrhiza pumila compounds (RPLC)
- 286 CDK chemical descriptors computed per compound (SMILES/structure input via Retip getCD())

## Outputs

- Five fitted, tuned Retip model objects (fit.rf, fit.brnn, fit.xgboost, fit.lightgbm, fit.keras)
- results_table.csv with MAE, RMSE, R2, and 95% confidence intervals for training, test, and validation predictions of all five models across both chromatographic methods
- Per-model [test-training] and [validation-training] MAE error differences used to rank overfitting

## How to apply

Load the HILIC training set (880 compounds from MoNA) and RPLC training set (398 compounds from the RIKEN PlaSMA database) with experimental retention times, and compute 286 CDK chemical descriptors per compound with Retip's getCD() built on rcdk. Split each library 80:20 with caret using XlogP as the index, center and scale descriptors with cesc(), and clean non-existent values and low-variance columns with proc.data(). Fit the five models with algorithm-specific tuning: fit.rf tunes mtry, fit.brnn tunes the number of neurons, fit.xgboost grid-searches nrounds, max_depth, and eta (fixed gamma, colsample_bytree, subsample, min_child_weight), fit.lightgbm picks the optimal nrounds by best iteration, and fit.keras tunes dense_unit, epochs, and dropout with validation_split=0.2; all non-Keras models use 10-fold cross-validation via caret. Predict test (143 HILIC / 96 RPLC compounds) and external validation sets (377 Pathogen Box HILIC, 358 Ophiorrhiza pumila RPLC compounds) with RT.spell(), then compute MAE, RMSE, R2, and 95% CIs with get.score(). Select the most robust model as the one with the smallest average [test-training] and [validation-training] MAE difference — in the paper Keras (0.14 min) beat RF (0.22), BRNN (0.33), XGBoost (0.39), and LightGBM (0.51).

## Related tools

- **Retip** (R package supplying getCD(), proc.data(), RT.spell(), and the fit.* model-fitting functions used for tuning and prediction) — https://github.com/olobion/Retip
- **caret** (80:20 data splitting (indexed on XlogP) and 10-fold cross-validation for the non-Keras models)
- **Keras** (Tuned via dense_unit, epochs, dropout with validation_split=0.2; best-performing model with MAE 0.78 min (HILIC) and 0.57 min (RPLC))
- **XGBoost** (Grid search over nrounds, max_depth, eta with fixed gamma, colsample_bytree, subsample, min_child_weight)
- **LightGBM** (Optimal nrounds selected automatically via best iteration)
- **rcdk** (Computes the 286 CDK chemical descriptors that serve as tuning features)

## Examples

```
devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Reproduced Keras test MAE matches approximately 0.78 min for HILIC and 0.57 min for RPLC
- Keras average error difference across test/validation vs training is about 0.14 min and is the smallest of the five models
- Per-model average error differences reproduce the reported ranking: RF 0.22, BRNN 0.33, XGBoost 0.39, LightGBM 0.51 min
- results_table.csv contains metrics (MAE, RMSE, R2, 95% CI) for all five models across both HILIC and RPLC methods, each with training, test, and validation rows
- Data split is 80:20 with XlogP as the caret index, and non-Keras models report 10-fold cross-validation

## Limitations

- 1 min prediction errors preclude discriminating closely eluting compounds in typical 12 min UPLC methods
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity
- Molecular descriptors may not fully explain complex interactions between LC buffer systems (including ionic strengths) and microenvironments, so no current models yield validation MAEs better than ±1 min
- Retention-time predictions were not tested for adjustment to small changes in chromatographic methods

## Evidence

- [methods] keras-tuning: [evidence span withheld — anchor status: not_found]
