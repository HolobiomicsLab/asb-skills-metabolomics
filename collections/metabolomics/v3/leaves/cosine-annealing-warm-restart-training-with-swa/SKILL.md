---
name: cosine-annealing-warm-restart-training-with-swa
description: Use when training a deep neural network regressor on high-dimensional
  molecular features (e.g., alvaDesc fingerprints/descriptors) where a heavily regularized
  model must generalize, and target MAE/MEDAE benchmarks exist to reproduce.
when_to_use_negative:
- The task is a classification or annotation ranking problem rather than RT regression
  (use the GP meta-learning projection machinery instead)
- Only a handful of molecules with experimental RTs are available for a new chromatographic
  method — the DNN needs the full 80,038-record SMRT training set
- Features are already a small, low-dimensional curated set where simpler regressors
  (XGBoost, LightGBM, CatBoost) suffice
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: cmmrt
  role: Python package implementing the DNN regressor with cosine annealing warm restarts
    and SWA, plus nested-CV training/evaluation scripts
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the 5,666 molecular descriptors and 2,214 fingerprints (MACCS166,
    ECFP, PFP) that form the DNN input features
- name: Python
  role: Language in which the cmmrt package and training scripts are implemented and
    run
  repo: https://github.com/constantino-garcia/cmmrt
- name: RDKit
  role: Alternative fingerprint generator used in train_with_rdkit.ipynb to train
    the same DNN without an alvaDesc license
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:54e70280c9a4ffd98f364454646f9473b79b1275c4e28656869b8a5ce6d82098
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - RDKit
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Cosine annealing warm restart training with SWA

## Summary

A DNN training recipe that combines cosine annealing warm restarts of the learning rate with Stochastic Weight Averaging (SWA) to train a heavily regularized neural network for regression tasks such as retention-time prediction, yielding the best model in the CMM-RT paper (MAE 39.2±1.2 s, MEDAE 17.2±0.9 s on SMRT).

## When to use

Use when training a deep neural network regressor on high-dimensional molecular features (e.g., alvaDesc fingerprints/descriptors) where a heavily regularized model must generalize, and target MAE/MEDAE benchmarks exist to reproduce.

## When NOT to use

- The task is a classification or annotation ranking problem rather than RT regression (use the GP meta-learning projection machinery instead)
- Only a handful of molecules with experimental RTs are available for a new chromatographic method — the DNN needs the full 80,038-record SMRT training set
- Features are already a small, low-dimensional curated set where simpler regressors (XGBoost, LightGBM, CatBoost) suffice

## Inputs

- Fingerprint/descriptor feature matrix (5,666 alvaDesc descriptors and/or 2,214 MACCS166/ECFP/PFP fingerprint bits) for 80,038 SMRT molecules
- Experimental retention-time targets (seconds, reversed-phase LC-MS)
- Train/test split or cross-validation fold indices

## Outputs

- Trained DNN regressor checkpoint saved to a models folder (e.g., saved_models)
- Outer-fold predictions aggregated into MAE and MEDAE (mean ± standard error) for all, retained, and non-retained (RT < 300 s) molecules
- Optuna study results stored in a SQLite DB (e.g., results/optuna/train.db) and CV summary CSV (e.g., results/rt/rt_cv.csv)

## How to apply

Build a DNN regressor with 3 hidden layers and large dropout rates, applied to preprocessed features (standardized, median-imputed descriptors, low-variance fingerprint bits removed). Train with a cosine annealing warm-restart learning-rate schedule followed by Stochastic Weight Averaging over the annealing cycles. Tune hidden-layer sizes, dropout rates, activations, and restart/annealing lengths with a TPE Bayesian search (e.g., Optuna) maximizing negative MEDAE over 50 trials. Apply a quantile transformation to the RT targets before fitting, then evaluate via nested stratified cross-validation (5 outer folds, 7 inner folds, RT binned into 6 strata) and report mean/median absolute errors as mean ± standard error of outer-fold predictions.

## Related tools

- **cmmrt** (Python package implementing the DNN regressor with cosine annealing warm restarts and SWA, plus nested-CV training/evaluation scripts) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, ECFP, PFP) that form the DNN input features)
- **Python** (Language in which the cmmrt package and training scripts are implemented and run) — https://github.com/constantino-garcia/cmmrt
- **RDKit** (Alternative fingerprint generator used in train_with_rdkit.ipynb to train the same DNN without an alvaDesc license)

## Examples

```
make test_predictor  # remove --smoke_test in Makefile for full training: python cmmrt/rt/train_model.py --storage sqlite:///results/optuna/train.db --save_to saved_models
```

## Evaluation signals

- Outer-loop predictions reproduce the paper's fingerprint-DNN benchmark: MAE ≈ 39.2 ± 1.2 s and MEDAE ≈ 17.2 ± 0.9 s on all molecules (comparable to Tables 1 and Figs. 5/S3)
- Median absolute error is far lower than mean absolute error, confirming handling of the heavy-tailed RT distribution including non-retained molecules (RT < 300 s)
- Optuna study records ~50 TPE trials per inner fold maximizing negative MEDAE in results/optuna/cv.db
- Removing the --smoke_test flag from the Makefile rule yields full-dataset training (80,038 records) rather than the subsampled smoke test
- Stratification of the RT target into 6 bins produces stable performance across the 5 outer folds

## Limitations

- Results are for reversed-phase LC-MS RTs; predicted RTs must be projected to other chromatographic methods via the Bayesian meta-learning GP before use
- alvaDesc is licensed software required to generate the original fingerprint/descriptor features (RDKit fingerprints are a substitute but not identical)
- Makefile rules run only smoke tests by default; full training requires removing the --smoke_test line and can take hours
- The repository's `paper` branch should be used as reference for reproducing the published results, as the code continues to evolve

## Evidence

- [abstract] The best results were obtained by a heavily regularized deep neural network trained with cosine annealing warm restarts and stochastic weight averaging, achieving a mean and median absolute errors: "The best results were obtained by a heavily regularized deep neural network trained with cosine annealing warm restarts and stochastic weight averaging, achieving a mean and median absolute errors"
- [methods] cosine annealing warm restarts: "we used cosine annealing warm restarts [39]."
- [methods] quantile transformation was applied to RTs before fitting.: "quantile transformation was applied to RTs before fitting."
- [methods] Highly correlated features were also eliminated (correlation >0.9): [evidence span withheld — anchor status: not_found]
- [readme] test_predictor      Test the performance of all RT predictors using nested cross-validation: "# test_predictor      Test the performance of all RT predictors using nested cross-validation"
