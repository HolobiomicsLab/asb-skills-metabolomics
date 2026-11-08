---
name: overfitting-assessment-via-error-differences
description: Use when several regression models (e.g. RF, BRNN, XGBoost, LightGBM,
  Keras) have been fit on the same training data with 80:20 splits and you must decide
  which model generalizes by comparing [test-training] and [validation-training] MAE
  gaps rather than raw test-set metrics alone.
when_to_use_negative:
- You only have a single train/test split with no external validation set — the [validation-training]
  gap, which is the core signal here, cannot be computed
- The task is model selection by lowest absolute error only, not generalization robustness
  — overfitting differences can invert the ranking (LightGBM's 0.51 min gap matters
  only if robustness is the criterion)
- Predictions are requested for compounds outside the chemical landscape of the training
  set (e.g. complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines,
  which were excluded from the Retip training data)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: Retip
  role: Provides get.score() for computing MAE, RMSE, R2 and 95% confidence intervals
    on training/test/validation predictions, and RT.spell() for generating the predictions
    the gaps are computed on
  repo: https://github.com/olobion/Retip
- name: R
  role: Runtime environment for the Retip package in which the model comparison workflow
    is executed
  repo: https://www.retip.app
- name: caret
  role: Used for the 80:20 data split (indexed by XlogP) and 10-fold cross-validation
    during model tuning, which is the prerequisite for a fair train-vs-test error
    comparison
- name: Keras
  role: The neural-network model (tuned dense_unit, epochs, dropout) that showed the
    smallest error difference and is therefore the reference winner of this assessment
- name: XGBoost
  role: Baseline model whose [test-training] and [validation-training] gaps (0.39
    min average) are compared against the best model
- name: LightGBM
  role: Baseline gradient-boosting model with the largest error difference (0.51 min
    average), used as the anti-pattern in the comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:c7f87716c948f47001f504ebc5e2589a156efa51c93a9f74dfd63fb5b897c87a
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# overfitting-assessment-via-error-differences

## Summary

Assesses overfitting of regression models by comparing MAE differences between training vs test and training vs external validation sets, selecting the model with the smallest error differences. In the Retip retention-time study this identified Keras as the most robust of five ML models (average error difference 0.14 min vs 0.22–0.51 min for RF, BRNN, XGBoost, LightGBM).

## When to use

Use when several regression models (e.g. RF, BRNN, XGBoost, LightGBM, Keras) have been fit on the same training data with 80:20 splits and you must decide which model generalizes best by comparing [test-training] and [validation-training] MAE gaps rather than raw test-set metrics alone.

## When NOT to use

- You only have a single train/test split with no external validation set — the [validation-training] gap, which is the core signal here, cannot be computed
- The task is model selection by lowest absolute error only, not generalization robustness — overfitting differences can invert the ranking (LightGBM's 0.51 min gap matters only if robustness is the criterion)
- Predictions are requested for compounds outside the chemical landscape of the training set (e.g. complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines, which were excluded from the Retip training data)

## Inputs

- Trained regression model objects (fit.rf, fit.brnn, fit.xgboost, fit.lightgbm, fit.keras)
- Training, test and external validation prediction sets with experimental retention times (minutes)
- Per-set MAE values from Retip get.score() for training, test, and validation predictions

## Outputs

- Per-model [test-training] and [validation-training] MAE error differences
- Average error-difference ranking across models (e.g. Keras 0.14, RF 0.22, BRNN 0.33, XGBoost 0.39, LightGBM 0.51 min)
- Full metrics table (MAE, RMSE, R2, 95% CI per split per model) written to results_table.csv

## How to apply

Fit each candidate model on the training split (e.g. 880 HILIC compounds from MoNA and 398 RPLC compounds from RIKEN PlaSMA, with 286 CDK descriptors computed via Retip getCD()), tuning hyperparameters with 10-fold cross-validation via caret (Keras uses validation_split=0.2). Predict on the held-out test set (143 HILIC / 96 RPLC compounds) and, critically, on an independent external validation set (377 Pathogen Box compounds for HILIC; 358 Ophiorrhiza pumila compounds for RPLC). For each model compute MAE, RMSE, R2 and 95% CIs with Retip's get.score() on training, test and validation predictions. Calculate the [test-training] and [validation-training] MAE differences and average them per model; a small average difference indicates minimal overfitting. Rank models by this average gap — in the reference reproduction Keras scored 0.14 min, ahead of RF 0.22, BRNN 0.33, XGBoost 0.39, and LightGBM 0.51 min — and prefer the model with both a small gap and low absolute MAE (Keras: 0.78 min HILIC, 0.57 min RPLC).

## Related tools

- **Retip** (Provides get.score() for computing MAE, RMSE, R2 and 95% confidence intervals on training/test/validation predictions, and RT.spell() for generating the predictions the gaps are computed on) — https://github.com/olobion/Retip
- **R** (Runtime environment for the Retip package in which the model comparison workflow is executed) — https://www.retip.app
- **caret** (Used for the 80:20 data split (indexed by XlogP) and 10-fold cross-validation during model tuning, which is the prerequisite for a fair train-vs-test error comparison)
- **Keras** (The neural-network model (tuned dense_unit, epochs, dropout) that showed the smallest error difference and is therefore the reference winner of this assessment)
- **XGBoost** (Baseline model whose [test-training] and [validation-training] gaps (0.39 min average) are compared against the best model)
- **LightGBM** (Baseline gradient-boosting model with the largest error difference (0.51 min average), used as the anti-pattern in the comparison)

## Examples

```
score <- get.score(pred.train, pred.test, pred.val); diff <- c(mean(abs(mae.test - mae.train)), mean(abs(mae.val - mae.train))); write.table(metrics, 'results_table.csv', sep=',')
```

## Evaluation signals

- The reproduced Keras test MAE matches ~0.78 min for HILIC and ~0.57 min for RPLC
- Keras average error difference across test and validation is ~0.14 min and is the smallest of the five models, with the reported ranking intact (RF 0.22, BRNN 0.33, XGBoost 0.39, LightGBM 0.51 min)
- The metrics table contains MAE, RMSE, R2, and 95% CI for all five models on all three splits (training, test, validation) for both HILIC and RPLC
- results_table.csv is produced and every [validation-training] MAE difference is a non-negative, interpretable value in minutes
- External validation sets of the correct sizes (377 Pathogen Box for HILIC, 358 Ophiorrhiza pumila for RPLC) were scored, not just the internal test split

## Limitations

- None of the current models yield validation MAEs better than ±1 min in retention prediction, suggesting molecular descriptors themselves are not sufficiently complex to fully explain LC buffer-system and adsorption/desorption interactions
- A ~1 min error window excludes about 25% of true positives and precludes discriminating closely eluting compounds in typical 12 min UPLC methods
- Prediction accuracy is limited by the number of training compounds and their chemical diversity; predictions outside the training chemical landscape may be more erroneous
- Many chemicals in the test and validation sets were not well represented by the training models

## Evidence
