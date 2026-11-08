---
name: retention-time-regression-model-training
description: Use when experimental retention times are available for a compound library
  (e.g. 880 HILIC compounds from MoNA or 398 RPLC compounds from RIKEN PlaSMA) and
  the goal is a descriptor-based RT prediction model to filter false-positive annotations.
when_to_use_negative:
- When predictions will be made for compounds far outside the chemical landscape of
  the training set — the authors state 'if Retip is used for predictions outside the
  chemical landscape of the training set, results may be more erroneous'
- When the goal is to discriminate closely eluting compounds in typical ~12 min UPLC
  methods, since ~1 min prediction errors preclude narrow-window separation
- When the chromatographic method has small deviations from the training method —
  the study explicitly did not test adjusting retention predictions for small changes
  in chromatographic methods
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Retip
  role: R package providing getCD(), RT.spell(), get.score(), proc.data(), cesc()
    and the model-fitting wrappers for the full RT prediction workflow
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: Computes the 286 CDK chemical molecular descriptors used as regression features
- name: caret
  role: 80:20 data splitting (indexed on XlogP) and 10-fold cross-validation for the
    non-Keras models
- name: Keras
  role: Best-performing neural network model; tuned via dense_unit, epochs, dropout
    with validation_split=0.2
- name: XGBoost
  role: Gradient-boosting model tuned by grid search over nrounds, max_depth, eta
- name: LightGBM
  role: Gradient-boosting model fit with automatic optimal nrounds via best iteration
- name: randomForest
  role: Random forest baseline tuned on mtry
- name: brnn
  role: Bayesian-regularized neural network baseline tuned on neuron count
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:d82c87e77f205f461b5ab031c21033c806aca18514ea3baab0d3fd4b4578428f
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
  - brnn
  - caret
  - randomForest
  - rcdk
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# retention-time-regression-model-training

## Summary

Train and benchmark machine-learning regression models (RF, BRNN, XGBoost, LightGBM, Keras) that predict LC retention times from 286 CDK chemical descriptors, using the Retip R package for HILIC and reversed-phase metabolomics libraries, and select the model with the smallest overfitting gap between training, test, and validation MAE.

## When to use

Use when experimental retention times are available for a compound library (e.g. 880 HILIC compounds from MoNA or 398 RPLC compounds from RIKEN PlaSMA) and the goal is a descriptor-based RT prediction model to filter false-positive annotations.

## When NOT to use

- When predictions will be made for compounds far outside the chemical landscape of the training set — the authors state 'if Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous'
- When the goal is to discriminate closely eluting compounds in typical ~12 min UPLC methods, since ~1 min prediction errors preclude narrow-window separation
- When the chromatographic method has small deviations from the training method — the study explicitly did not test adjusting retention predictions for small changes in chromatographic methods

## Inputs

- HILIC training set: 880 compounds from MoNA with experimental retention times
- RPLC training set: 398 compounds from the RIKEN PlaSMA database with experimental retention times
- SMILES/compound structures for descriptor calculation via getCD()
- External validation sets (e.g. 377 Pathogen Box compounds for HILIC, 358 Ophiorrhiza pumila compounds for RPLC)

## Outputs

- results_table.csv with MAE, RMSE, R2, and 95% confidence intervals for training, test, and validation predictions of all five models on both HILIC and RPLC
- Per-model overfitting metrics: [test-training] and [validation-training] MAE error differences
- Trained retention time prediction models usable downstream in MS-DIAL/MS-FINDER candidate filtering

## How to apply

Compute 286 CDK chemical descriptors for each compound with Retip's getCD() (built on rcdk), split the library 80:20 with caret using XlogP as the stratification index, then clean non-existent values and low-variance columns with proc.data() and center/scale with cesc(). Fit five tuned models: fit.rf (tuning mtry), fit.brnn (neuron count), fit.xgboost (grid over nrounds, max_depth, eta with fixed gamma, colsample_bytree, subsample, min_child_weight), fit.lightgbm (best-iteration nrounds), and fit.keras (tuned dense_unit, epochs, dropout with validation_split=0.2); the non-Keras models use 10-fold cross-validation. Predict on held-out test sets (143 HILIC / 96 RPLC compounds) and external validation sets (377 Pathogen Box / 358 Ophiorrhiza pumila compounds) with RT.spell(), then compute MAE, RMSE, R2, and 95% CIs per set with get.score() and the [test-training] and [validation-training] MAE differences to assess overfitting. Write all metrics for both chromatographic methods and all models to results_table.csv; pick the model with the smallest average error difference (Keras at ~0.14 min in the paper, vs RF 0.22, BRNN 0.33, XGBoost 0.39, LightGBM 0.51).

## Related tools

- **Retip** (R package providing getCD(), RT.spell(), get.score(), proc.data(), cesc() and the model-fitting wrappers for the full RT prediction workflow) — https://github.com/olobion/Retip
- **rcdk** (Computes the 286 CDK chemical molecular descriptors used as regression features)
- **caret** (80:20 data splitting (indexed on XlogP) and 10-fold cross-validation for the non-Keras models)
- **Keras** (Best-performing neural network model; tuned via dense_unit, epochs, dropout with validation_split=0.2)
- **XGBoost** (Gradient-boosting model tuned by grid search over nrounds, max_depth, eta)
- **LightGBM** (Gradient-boosting model fit with automatic optimal nrounds via best iteration)
- **randomForest** (Random forest baseline tuned on mtry)
- **brnn** (Bayesian-regularized neural network baseline tuned on neuron count)

## Examples

```
devtools::install_github('olobion/Retip')
```

## Evaluation signals

- Reproduced Keras test MAE matches approximately 0.78 min for HILIC and 0.57 min for RPLC
- Keras average error difference across test/validation vs training is about 0.14 min and is the smallest of the five models (ranking: RF 0.22, BRNN 0.33, XGBoost 0.39, LightGBM 0.51 min)
- Validation MAEs are near the reported ~1 min threshold used for excluding false positives
- results_table.csv contains complete MAE/RMSE/R2 rows with 95% confidence intervals for all five models on both HILIC and RPLC
- Descriptor preprocessing leaves no non-existent values or low-variance columns after proc.data() and cesc()

## Limitations

- 1 min prediction-error windows excluded about 25% of true positives, so RT filtering trades recall for specificity
- Molecular descriptors alone seem to fail to fully explain complex interactions between LC buffer systems (including ionic strengths) and microenvironments — none of the current models yield validation MAEs better than ±1 min
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity; complex high-molecular-weight lipids (triacylglycerols, phosphatidylcholines) were largely excluded from the training set
- Predictions are not adequate for narrow retention time search windows but are useful for disclosing likely in-source fragmentation MS/MS spectra and ranking positional isomers

## Evidence
