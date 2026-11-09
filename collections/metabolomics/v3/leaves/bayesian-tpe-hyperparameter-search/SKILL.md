---
name: bayesian-tpe-hyperparameter-search
description: Use when training retention-time (or other regression) models whose hyperparameters
  (DNN layer sizes, dropout, learning schedules, GBM depths) must be tuned to maximize
  median absolute error rather than MAE, before nested-CV evaluation.
when_to_use_negative:
- You already have fixed hyperparameters and only need final evaluation — running
  the TPE search would waste trials; use the test/nested-CV path instead.
- The task is meta-training the GP for RT projections between chromatographic methods
  (make train_projections) — that uses a different objective and the PredRet database,
  not the RT-predictor TPE search.
- Your targets or evaluation metric differ fundamentally (e.g., classification of
  non-retained vs retained molecules is a separate step, not what the TPE search optimizes).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: cmmrt
  role: Python package implementing the RT predictors and Optuna-driven TPE hyperparameter
    search; invoked via cmmrt/rt/train_model.py
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Runtime environment for the cmmrt package, Optuna storage, and cross-validation
    scripts
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the descriptors and fingerprints (ECFP, MACCSFP, PFP) whose model
    configuration the search tunes over
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:2bab646470f3a26ff4cdd4ce047b7b6d578b9e4cf650ad943d78f2e22c20b108
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - RDKit
  - alvaDesc
  - cmmrt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Bayesian TPE hyperparameter search

## Summary

Tunes RT-prediction regressors (hidden-layer sizes, dropout rates, activations, restart/annealing lengths) with a TPE Bayesian search (via Optuna) maximizing the negative median absolute error (MEDAE) on the SMRT dataset, then evaluates tuned models with nested cross-validation.

## When to use

Use when training retention-time (or other regression) models whose hyperparameters (DNN layer sizes, dropout, learning schedules, GBM depths) must be tuned to maximize median absolute error rather than MAE, before nested-CV evaluation.

## When NOT to use

- You already have fixed hyperparameters and only need final evaluation — running the TPE search would waste trials; use the test/nested-CV path instead.
- The task is meta-training the GP for RT projections between chromatographic methods (make train_projections) — that uses a different objective and the PredRet database, not the RT-predictor TPE search.
- Your targets or evaluation metric differ fundamentally (e.g., classification of non-retained vs retained molecules is a separate step, not what the TPE search optimizes).

## Inputs

- SMRT dataset: 80,038 experimental retention times (reversed-phase LCMS) with pubchem id / InChI / SMILES compound identifiers
- Preprocessed feature matrices: 5,666 alvaDesc molecular descriptors and 2,214 fingerprint bits (ECFP, MACCS166, PFP)
- Quantile-transformed RT target values

## Outputs

- Optuna study database of hyperparameter search results (e.g., results/optuna/train.db, nested results/optuna/cv.db)
- Tuned model artifacts saved to saved_models (preprocessor + regressor/blender)
- Nested cross-validation summary CSV (e.g., results/rt/rt_cv.csv) with MAE and MEDAE (mean ± standard error)

## How to apply

Build the model candidates (e.g., a heavily regularized DNN with cosine annealing warm restarts and stochastic weight averaging) on preprocessed SMRT features: standardized descriptors with median imputation and missing indicators, zero-variance features removed, features correlated >0.9 eliminated, low-variance fingerprint bits removed, and a quantile transformation applied to RTs before fitting. Run a TPE Bayesian search maximizing negative MEDAE for ~50 trials, persisting every trial to a SQLite study database (e.g., results/optuna/train.db) via Optuna so partial runs are auditable. The tuned settings then feed a nested stratified cross-validation with 5 outer and 7 inner folds, stratifying the RT target into 6 bins so both retained and non-retained molecules (RT < 5 min) are represented. Use a fixed --random_state to make trials reproducible or reusable, and start with the Makefile's smoke_test to validate the wiring before full training. Aggregate outer-loop predictions into MAE and MEDAE (mean ± standard error) as the final benchmark.

## Related tools

- **cmmrt** (Python package implementing the RT predictors and Optuna-driven TPE hyperparameter search; invoked via cmmrt/rt/train_model.py) — https://github.com/constantino-garcia/cmmrt
- **Python** (Runtime environment for the cmmrt package, Optuna storage, and cross-validation scripts) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the descriptors and fingerprints (ECFP, MACCSFP, PFP) whose model configuration the search tunes over)

## Examples

```
python cmmrt/rt/train_model.py --storage sqlite:///results/optuna/train.db --save_to saved_models --trials 50 --random_state 42
```

## Evaluation signals

- The Optuna study DB (results/optuna/train.db) records ~50 trials with the objective value equal to negative MEDAE; the best trial's params correspond to valid DNN configs (hidden sizes, dropout rates, activations, restart/annealing lengths).
- Nested CV summary (results/rt/rt_cv.csv) reports MAE and MEDAE per outer fold; benchmark reproduction gives MAE ≈ 39.2 ± 1.2 s and MEDAE ≈ 17.2 ± 0.9 s on all SMRT molecules.
- Outer/inner fold counts are 5 and 7 with RT stratified into 6 bins; error breakdowns should be reported separately for all, retained, and non-retained molecules (RT < 300 s).
- Re-running with the same --random_state reuses/reproduces identical param search results from the SQLite study.
- Removing the --smoke_test flag and re-running should use all regressors and the complete 80,038-record SMRT dataset, with trial times increasing accordingly.

## Limitations

- Makefile rules run with --smoke_test (small models, subsampled data, overridden trials/folds); forgetting to remove it produces toy results, not paper-level numbers.
- The paper recommends not reusing hyperparameter search databases unless you understand partial runs — stale Optuna studies can silently bias the best-trial selection.
- alvaDesc is licensed software; without it you cannot regenerate the fingerprint/descriptor feature set (RDKit fingerprints are only offered as an alternative in train_with_rdkit.ipynb).
- Search optimizes negative MEDAE specifically; the best params may not be optimal for MAE or per-class (retained vs non-retained) performance.

## Evidence

- [methods] quantile transformation was applied to RTs before fitting.: "quantile transformation was applied to RTs before fitting."
- [methods] cosine annealing warm restarts: "we used cosine annealing warm restarts [39]."
- [readme] Bayesian hyperparameter search on SMRT subset: "Trains a subset of regressors on a subset of the SMRT database using Bayesian hyperparameter search."
- [readme] Optuna trial persistence: "A summary of all tested models is stored in the database `results/optuna/train.db`."
