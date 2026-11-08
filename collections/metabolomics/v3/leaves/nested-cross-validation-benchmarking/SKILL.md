---
name: nested-cross-validation-benchmarking
description: Use when you must compare several hyperparameter-tuned regressors on
  one labeled dataset (e.g. 80,038 SMRT retention times with alvaDesc descriptors/fingerprints)
  and need an unbiased error estimate without validation-set leakage.
when_to_use_negative:
- You only need to train and deploy a single model, not compare several regressors
  — nested CV adds large compute cost for no comparison benefit.
- Your dataset is too small to afford 5x7 cross-validation folds (the paper relies
  on 80,038 labeled molecules).
- Your evaluation design already isolates a fully held-out external test set, making
  nested resampling redundant for unbiased selection.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: cmmrt
  role: Python package implementing the RT regressors, blending ensemble, nested cross-validation
    and Optuna hyperparameter search
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the 5,666 molecular descriptors and 2,214 ECFP/MACCS166/PFP fingerprints
    used as regressor input features
  repo: https://www.alvascience.com/alvadesc/
- name: XGBoost
  role: Benchmark regressor and non-retained binary classifier (maximize F1, fingerprint
    features only)
- name: LightGBM
  role: Benchmark regressor tuned stepwise for speed
- name: CatBoost
  role: Benchmark regressor with class weights on non-retained molecules varied from
    1e-6 to 80
- name: Python
  role: Implementation language; code released via the cmmrt GitHub repository
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:6151f82a272e36bf84d5911865307550d46f9e7fec07649a4c55a343de404227
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
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nested-cross-validation-benchmarking

## Summary

Benchmark protocol for comparing multiple machine learning regressors (XGBoost, LightGBM, CatBoost, DKL, regularized DNN) on a regression task such as retention-time prediction, using nested stratified cross-validation with Bayesian hyperparameter search and an optional blending ensemble to avoid optimistically biased, leaky model comparisons.

## When to use

Use when you must compare several hyperparameter-tuned regressors on one labeled dataset (e.g. 80,038 SMRT retention times with alvaDesc descriptors/fingerprints) and need an unbiased error estimate without validation-set leakage.

## When NOT to use

- You only need to train and deploy a single model, not compare several regressors — nested CV adds large compute cost for no comparison benefit.
- Your dataset is too small to afford 5x7 cross-validation folds (the paper relies on 80,038 labeled molecules).
- Your evaluation design already isolates a fully held-out external test set, making nested resampling redundant for unbiased selection.

## Inputs

- SMILES/SDF/mol/mol2/hin structure files or a CSV with pubchem id and inchi/SMILES for the 80,038-molecule SMRT dataset
- Feature matrices of alvaDesc descriptors (standardized, median-imputed) and ECFP/MACCS166/PFP fingerprints (zero-variance and >0.9-correlation features removed)
- Numerical target vector of experimental retention times in seconds
- Optional binary non-retained labels (RT < 5 minutes) from an XGBoost classifier trained on fingerprint features

## Outputs

- Comparison summary CSV (results/rt/rt_cv.csv) with per-fold MAE and MEDAE for all molecules, non-retained, and retained subsets, grouped by regressor family and feature set
- Optuna hyperparameter-search databases (results/optuna/train.db and nested results/optuna/cv.db)
- Trained blending ensemble (best blender) with base-regressor models saved under saved_models/

## How to apply

Split the feature sets (fingerprints, descriptors, or both, computed with alvaDesc: 5,666 descriptors and 2,214 ECFP/MACCS166/PFP fingerprints) after standardization, median imputation with a missing indicator, zero-variance removal, and dropping features correlated >0.9. Run nested stratified cross-validation with 5 outer and 7 inner folds, stratifying the target into 6 bins, and tune hyperparameters with Bayesian TPE (e.g. 50 iterations via Optuna) maximizing negative median absolute error (MEDAE) on inner validation splits so hyperparameter search never sees outer-fold test data. Train candidates such as XGBoost, weighted CatBoost (non-retained weights varied 1e-6 to 80 at ratio 1/40), LightGBM, DKL, and DNNs with quantile-transformed RTs, cosine annealing warm restarts, and stochastic weight averaging. For the blending ensemble, hold out 20% of training data, store base-regressor predictions on it, and fit a Bayesian-tuned random-forest meta-regressor. Aggregate MAE and MEDAE across folds for all, non-retained (RT < 5 min / 300 s), and retained subsets into a comparison CSV grouped by regressor family and feature set. Note that the paper's best result — MAE 39.2 ± 1.2 s and MEDAE 17.2 ± 0.9 s — came from the heavily regularized DNN, and fingerprints tended to outperform descriptors alone.

## Related tools

- **cmmrt** (Python package implementing the RT regressors, blending ensemble, nested cross-validation and Optuna hyperparameter search) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the 5,666 molecular descriptors and 2,214 ECFP/MACCS166/PFP fingerprints used as regressor input features) — https://www.alvascience.com/alvadesc/
- **XGBoost** (Benchmark regressor and non-retained binary classifier (maximize F1, fingerprint features only))
- **LightGBM** (Benchmark regressor tuned stepwise for speed)
- **CatBoost** (Benchmark regressor with class weights on non-retained molecules varied from 1e-6 to 80)
- **Python** (Implementation language; code released via the cmmrt GitHub repository) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make test_predictor   # nested-CV benchmark; remove --smoke_test in cmmrt/rt/train_model.py for the full 80,038-molecule run
```

## Evaluation signals

- results/rt/rt_cv.csv exists and contains MAE and MEDAE rows aggregated per regressor family and per feature set (fingerprints, descriptors, fingerprints+descriptors) for all/non-retained/retained subsets.
- Inner-fold model selection maximizes negative MEDAE on validation splits only; outer-fold test data never influences hyperparameter search (verify by inspecting results/optuna/cv.db nested within the outer loop).
- A correctly reproduced run should recover the paper's ordering: heavily regularized DNN best, with MAE ≈ 39.2 ± 1.2 s and MEDAE ≈ 17.2 ± 0.9 s on SMRT.
- The blending ensemble holds out exactly 20% of training data before fitting the random-forest meta-regressor (train_size parameter controls this split).
- RT stratification uses 6 bins with 5 outer and 7 inner stratified folds, so per-fold class/bin counts should be consistent across folds.

## Limitations

- The Makefile runs only smoke tests (subsampled data, reduced trials); full reproduction requires removing the --smoke_test flag and waiting hours, and reusing partial hyperparameter search databases is discouraged.
- Fingerprint generation depends on the licensed alvaDesc software; only the RDKit-fingerprint notebook (train_with_rdkit.ipynb) avoids that dependency and may not match paper results.
- Only the `paper` branch of the cmmrt repository reproduces the published results, as the main branch continues to be developed toward CEU Mass Mediator integration.
- Unlike the original SMRT evaluation, this benchmark includes non-retained molecules, so error numbers are not directly comparable to earlier SMRT papers.

## Evidence

- [readme] Tests the performance of a subset of regressors on a subset of the SMRT database using nested cross-validation. A summary of the results is stored in the CSV `results/rt/rt_cv.csv`.: "Tests the performance of a subset of regressors on a subset of the SMRT database using nested cross-validation. A summary of the results is stored in the CSV `results/rt/rt_cv.csv`."
- [readme] The best results were obtained by a heavily regularized DNN trained with cosine annealing warm restarts and stochastic weight averaging, achieving a mean and median absolute errors of 39.2±1.2 s and 17.2 ± 0.9 s, respectively.: "The best results were obtained by a heavily regularized DNN trained with cosine annealing warm restarts and stochastic weight averaging, achieving a mean and median absolute errors of 39.2±1.2 s and"
- [abstract] 5, 666 molecular descriptors and 2, 214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software: "5, 666 molecular descriptors and 2, 214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with the alvaDesc software"
- [methods] a molecule was considered non-retained if its Retention Time (RT) was smaller than 5 minutes.: "a molecule was considered non-retained if its Retention Time (RT) was smaller than 5 minutes."
- [intro] In the evaluation presented in [21] the molecules that were not retained by the column were excluded. In our evaluation, both retained and non-retained molecules will be considered.: "In the evaluation presented in [21] the molecules that were not retained by the column were excluded. In our evaluation, both retained and non-retained molecules will be considered."
