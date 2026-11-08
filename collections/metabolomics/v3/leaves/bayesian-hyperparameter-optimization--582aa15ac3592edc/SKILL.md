---
name: bayesian-hyperparameter-optimization
description: Use when training RT-prediction regressors (XGBoost, LightGBM, CatBoost,
  DKL, DNN) on alvaDesc fingerprints/descriptors and you need tuned hyperparameters
  with leakage-free nested-CV error estimates.
when_to_use_negative:
- The task is RT projection between chromatographic methods (PredRet meta-learned
  GP workflow) rather than regressor training — that is a separate skill in this paper
- The compounds lack SMILES/SDF structures to feed alvaDesc, so no fingerprints or
  descriptors can be generated
- You only need the published best model's performance rather than re-tuning — the
  paper branch already reports MAE 39.2 ± 1.2 s and MEDAE 17.2 ± 0.9 s
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0092
tools:
- name: cmmrt
  role: Package implementing train_model.py / test_model.py for Bayesian-tuned RT
    regressors, blending ensemble, and nested CV; installed via make install
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Language in which the cmmrt package and its Optuna-based hyperparameter search
    are implemented and invoked
  repo: https://github.com/constantino-garcia/cmmrt
- name: XGBoost
  role: Base regressor tuned with 50 TPE iterations maximizing negative MEDAE; also
    provides the non-retained classifier feature
- name: LightGBM
  role: Base regressor tuned stepwise for speed within the same Bayesian search framework
- name: CatBoost
  role: Base regressor tuned with weighted non-retained molecules (weights 1e-6 to
    80, ratio 1/40); also run with defaults
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:f6012bd1cb3a52d98e30703ba4bee8adfb7a04a30bde2bd109c54b3d38d5dec0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - CatBoost
  - LightGBM
  - Python
  - XGBoost
  - alvaDesc
  - cmmrt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bayesian-hyperparameter-optimization

## Summary

Bayesian (TPE) hyperparameter search for retention-time prediction regressors on the 80,038-molecule SMRT dataset, run with 50 trials maximizing negative MEDAE inside nested stratified cross-validation so tuning never leaks into outer-fold error estimates.

## When to use

Use when training RT-prediction regressors (XGBoost, LightGBM, CatBoost, DKL, DNN) on alvaDesc fingerprints/descriptors and you need tuned hyperparameters with leakage-free nested-CV error estimates.

## When NOT to use

- The task is RT projection between chromatographic methods (PredRet meta-learned GP workflow) rather than regressor training — that is a separate skill in this paper
- The compounds lack SMILES/SDF structures to feed alvaDesc, so no fingerprints or descriptors can be generated
- You only need the published best model's performance rather than re-tuning — the paper branch already reports MAE 39.2 ± 1.2 s and MEDAE 17.2 ± 0.9 s

## Inputs

- SMRT dataset with 80,038 experimental retention times (reversed-phase LC-MS)
- Feature matrices built with alvaDesc: 5,666 molecular descriptors and 2,214 fingerprints (ECFP, MACCS166, Path Fingerprints), alone and combined
- Predicted non-retained binary feature (RT < 5 minutes / 300 s) from an XGBoost classifier on fingerprint features
- Preprocessed features: standardization, median imputation with missing indicator, zero-variance removal, removal of features correlated > 0.9

## Outputs

- Best blender (random-forest meta-regressor over all base regressors) saved to the saved_models folder
- SQLite Optuna databases recording all trials (results/optuna/train.db for training, results/optuna/cv.db nested within the outer CV loop)
- rt_cv.csv comparison summary aggregating MAE and MEDAE across folds, grouped by regressor family and feature set

## How to apply

Configure an Optuna study with a SQLite storage (e.g. sqlite:///results/optuna/train.db) and run 50 iterations of Bayesian TPE search maximizing negative MEDAE. Tune XGBoost, weighted CatBoost (non-retained class weights swept from 1e-6 to 80 with ratio 1/40), and DKL with the full search; tune LightGBM stepwise for speed and CatBoost with defaults; for DNNs apply a quantile transformation of RTs plus cosine annealing warm restarts with stochastic weight averaging. Wrap the search in nested stratified cross-validation with 5 outer and 7 inner folds, stratifying RTs into 6 bins and minimizing MEDAE on inner validation splits to avoid data leakage. Build the blending ensemble by holding out 20% of the training data, storing base-regressor predictions on it, and training a Bayesian-tuned random-forest meta-regressor. Do not reuse hyperparameter-search databases from partial runs unless you understand how to reuse them; start a fresh study instead.

## Related tools

- **cmmrt** (Package implementing train_model.py / test_model.py for Bayesian-tuned RT regressors, blending ensemble, and nested CV; installed via make install) — https://github.com/constantino-garcia/cmmrt
- **Python** (Language in which the cmmrt package and its Optuna-based hyperparameter search are implemented and invoked) — https://github.com/constantino-garcia/cmmrt
- **XGBoost** (Base regressor tuned with 50 TPE iterations maximizing negative MEDAE; also provides the non-retained classifier feature)
- **LightGBM** (Base regressor tuned stepwise for speed within the same Bayesian search framework)
- **CatBoost** (Base regressor tuned with weighted non-retained molecules (weights 1e-6 to 80, ratio 1/40); also run with defaults)

## Examples

```
python cmmrt/rt/train_model.py --storage sqlite:///results/optuna/train.db --save_to saved_models  # remove --smoke_test from the Makefile rule for full training
```

## Evaluation signals

- The Optuna SQLite database (results/optuna/train.db) records all trials, and the objective value is negative MEDAE, so lower validation MEDAE should correspond to higher objective values
- Nested CV is respected: results/optuna/cv.db is nested within the outer loop, and inner-fold tuning must never see outer-fold data
- rt_cv.csv exists and contains MAE/MEDAE rows aggregated across folds for all molecules plus retained and non-retained subsets, grouped by regressor family and feature set
- Reproducing the full run should recover the DNN with cosine annealing warm restarts and SWA as best regressor, at mean/median absolute errors of 39.2 ± 1.2 s and 17.2 ± 0.9 s
- The saved blender in saved_models contains all base regressors and the meta-regressor, trained with 20% of training data held out for the meta level

## Limitations

- Makefile rules run only smoke tests (--smoke_test uses small models and subsampled data, overriding param_search_folds and trials); the FIXME line must be removed for complete training
- Hyperparameter-search SQLite databases should not be reused from partial runs unless you really understand how to reuse them
- alvaDesc is required to generate fingerprints/descriptors and is only available under license

## Evidence

- [readme] Trains a subset of regressors on a subset of the SMRT database using Bayesian hyperparameter search. The best resulting blender (which contains all other regressors) is saved to the saved_models folder.: "Trains a subset of regressors on a subset of the SMRT database using Bayesian hyperparameter search. The best resulting blender (which contains all other regressors) is saved to the saved_models"
- [other] Tune and train XGBoost, weighted CatBoost (non-retained weights varied from 1e-6 to 80 with ratio 1/40), and DKL regressors with 50 iterations of Bayesian TPE hyperparameter search maximizing negative MEDAE: [evidence span withheld — anchor status: not_found]
- [other] Run nested stratified cross-validation with 5 outer and 7 inner folds, stratifying RTs into 6 bins and minimizing MEDAE on validation splits to avoid data leakage.: [evidence span withheld — anchor status: not_found]
- [readme] unless you really understand how to reuse partial runs, we recommend not reusing hyperparameter's databases.: "unless you really understand how to reuse partial runs, we recommend not reusing hyperparameter's databases."
- [abstract] The best results were obtained by a heavily regularized deep neural network trained with cosine annealing warm restarts and stochastic weight averaging, achieving a mean and median absolute errors: "The best results were obtained by a heavily regularized deep neural network trained with cosine annealing warm restarts and stochastic weight averaging, achieving a mean and median absolute errors"
