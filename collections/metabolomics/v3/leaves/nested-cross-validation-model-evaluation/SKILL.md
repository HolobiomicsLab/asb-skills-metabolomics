---
name: nested-cross-validation-model-evaluation
description: Use when comparing several ML regressors (e.g. DNN, XGBoost, LightGBM,
  CatBoost) on a labeled dataset where hyperparameter tuning must be separated from
  performance estimation to avoid optimistic bias.
when_to_use_negative:
- The goal is projecting RTs to a new chromatographic method with few known molecules
  — use the Bayesian meta-learning GP projection approach instead of full nested CV
  benchmarking
- Input is a metabolite annotation candidate list to be ranked — this skill only estimates
  predictor performance and does not score annotations
- You only need a quick sanity check of the pipeline — the README's --smoke_test flag
  exists for that and results will not match published metrics
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: cmmrt
  role: Python package implementing the nested-CV training/testing scripts (train_model.py,
    test_model.py), Optuna storage, and stratified fold utilities for SMRT RT regression
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Implementation language; all CV, tuning, and aggregation code is Python
  repo: https://github.com/constantino-garcia/cmmrt
- name: alvaDesc
  role: Generates the 5,666 descriptors and ECFP/MACCSFP/PFP fingerprints that form
    the model input matrix
  repo: https://www.alvascience.com/alvadesc/
- name: XGBoost
  role: One of the state-of-the-art regressors benchmarked against the DNN in the
    nested-CV comparison
- name: RDKit
  role: Alternative fingerprint source for training the DNN in train_with_rdkit.ipynb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:9a5903c492d926fe9b1d656e37dc76b7115921fb27ca0453a1c49b8b3976781a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - RDKit
  - XGBoost
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Nested cross-validation model evaluation

## Summary

Benchmark regression models (RT prediction on the 80,038-molecule METLIN SMRT dataset) with nested stratified cross-validation, using an inner TPE Bayesian hyperparameter search and unbiased outer-loop MAE/MEDAE estimates reported as mean ± standard error.

## When to use

Use when comparing several ML regressors (e.g. DNN, XGBoost, LightGBM, CatBoost) on a labeled dataset where hyperparameter tuning must be separated from performance estimation to avoid optimistic bias.

## When NOT to use

- The goal is projecting RTs to a new chromatographic method with few known molecules — use the Bayesian meta-learning GP projection approach instead of full nested CV benchmarking
- Input is a metabolite annotation candidate list to be ranked — this skill only estimates predictor performance and does not score annotations
- You only need a quick sanity check of the pipeline — the README's --smoke_test flag exists for that and results will not match published metrics

## Inputs

- SMRT dataset: 80,038 experimental retention-time records with PubChem IDs/InChI or SMILES structures
- alvaDesc feature matrix: 5,666 molecular descriptors and 2,214 fingerprint bits (ECFP, MACCS166, Path Fingerprints)
- Optionally RDKit fingerprints (as in train_with_rdkit.ipynb)
- Optuna sqlite storage DB for hyperparameter trials

## Outputs

- Outer-loop MAE and MEDAE (mean ± standard error) per model, stratified by all/retained/non-retained molecules, e.g. results/rt/rt_cv.csv
- Hyperparameter search summary per model in results/optuna/cv.db (nested within the outer loop)
- Trained/blended model artifacts saved under saved_models/

## How to apply

Preprocess features first: standardize alvaDesc descriptors, median-impute with missing indicators, drop zero-variance features and features correlated >0.9, remove low-variance fingerprint bits, and apply a quantile transformation to the RT targets before fitting. For each of 5 outer folds, run a TPE (Optuna-style) Bayesian search with 7 inner folds for 50 trials maximizing negative MEDAE, tuning hidden-layer sizes, dropout rates, activations and cosine-annealing warm-restart lengths. Stratify both inner and outer splits on the RT target binned into 6 bins so that rare short/non-retained RTs (RT < 5 min, i.e. < 300 s) appear in every fold. Aggregate outer-loop out-of-fold predictions to compute MAE and MEDAE (mean ± standard error) separately for all molecules, retained molecules, and non-retained molecules; unlike the original SMRT evaluation, non-retained molecules are kept in the benchmark. The paper's reference benchmark is a heavily regularized DNN (cosine annealing warm restarts + stochastic weight averaging) reaching MAE 39.2±1.2 s and MEDAE 17.2±0.9 s.

## Related tools

- **cmmrt** (Python package implementing the nested-CV training/testing scripts (train_model.py, test_model.py), Optuna storage, and stratified fold utilities for SMRT RT regression) — https://github.com/constantino-garcia/cmmrt
- **Python** (Implementation language; all CV, tuning, and aggregation code is Python) — https://github.com/constantino-garcia/cmmrt
- **alvaDesc** (Generates the 5,666 descriptors and ECFP/MACCSFP/PFP fingerprints that form the model input matrix) — https://www.alvascience.com/alvadesc/
- **XGBoost** (One of the state-of-the-art regressors benchmarked against the DNN in the nested-CV comparison)
- **RDKit** (Alternative fingerprint source for training the DNN in train_with_rdkit.ipynb)

## Examples

```
make test_predictor  # then remove --smoke_test from the Makefile for full nested CV on the complete SMRT dataset
```

## Evaluation signals

- Fingerprint-DNN benchmark reproduces MAE ≈ 39.2 ± 1.2 s and MEDAE ≈ 17.2 ± 0.9 s across the 5 outer folds (Tables 1, Figs. 5/S3)
- rt_cv.csv contains one row per tested regressor with MAE and MEDAE reported as mean ± standard error over outer folds
- Inner hyperparameter trials are stored in an Optuna DB nested within the outer loop (results/optuna/cv.db), confirming tuning happened inside each outer fold only
- Every outer and inner fold contains all 6 RT-target bins (stratification check), including non-retained molecules with RT < 300 s
- Separate MAE/MEDAE rows exist for all molecules, retained molecules, and non-retained molecules

## Limitations

- The Makefile rules run smoke tests (subsampled data, reduced trials/folds); the --smoke_test flag must be removed for full training and the published metrics will not match otherwise
- Hyperparameter search databases should not be reused across runs unless partial runs are deliberately reused, to avoid stale trials contaminating results
- alvaDesc requires a license and some descriptors may be missing for certain compounds, hence median imputation with missing indicators is required
- Complete training on all regressors and the full 80,038-molecule dataset takes hours

## Evidence

- [methods] Nested CV with quantile-transformed targets: "quantile transformation was applied to RTs before fitting."
- [methods] Non-retained molecule handling: "a molecule was considered non-retained if its Retention Time (RT) was smaller than 5 minutes."
- [intro] Evaluation includes non-retained molecules unlike original benchmark: "In the evaluation presented in [21] the molecules that were not retained by the column were excluded. In our evaluation, both retained and non-retained molecules will be considered."
- [abstract] Reference benchmark metrics: "The best results were obtained by a heavily regularized deep neural network trained with cosine annealing warm restarts and stochastic weight averaging"
- [readme] README: nested-CV testing rule and output: "Tests the performance of a subset of regressors on a subset of the SMRT database using nested cross-validation. A summary of the results is stored in the CSV `results/rt/rt_cv.csv`"
- [readme] README: smoke-test flag warning: "you may remove the last line for complete training, using all regressors and the complete SMRT dataset"
