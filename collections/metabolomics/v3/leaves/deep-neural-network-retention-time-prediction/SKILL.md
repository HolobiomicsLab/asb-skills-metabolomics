---
name: deep-neural-network-retention-time-prediction
description: Use when you need to predict LC-MS retention times (RTs) for small molecules
  from structural features (fingerprints/descriptors), including both retained and
  non-retained molecules (RT < 300 s), and want accuracy.
when_to_use_negative:
- You only have feature tables already computed and just need a simple baseline —
  gradient boosting models (XGBoost, LightGBM, CatBoost) are cheaper alternatives
  benchmarked in the same paper
- Your goal is projecting RTs between chromatographic methods from a few identified
  molecules — that requires the Bayesian meta-learned GP projection skill, not RT
  prediction
- alvaDesc license is unavailable and RDKit fingerprints are used — expect different
  feature sets (notebook train_with_rdkit.ipynb exists but does not reproduce the
  exact benchmark)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: alvaDesc
  role: Generates the 5,666 molecular descriptors and 2,214 ECFP/MACCSFP/PFP fingerprints
    used as DNN input features (commercial license required)
  repo: https://www.alvascience.com/alvadesc/
- name: cmmrt
  role: Python package implementing DNN training, hyperparameter search, and nested-CV
    evaluation of RT predictors
  repo: https://github.com/constantino-garcia/cmmrt
- name: RDKit
  role: Alternative fingerprint source for training the DNN without alvaDesc (see
    train_with_rdkit.ipynb)
- name: Python
  role: Implementation language; install via `make install` and run experiments through
    the Makefile
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:93c5959dc9c1015c6e2611f65f1539e56c7257476d847cc04d012ecf9b8432b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - RDKit
  - alvaDesc
  - cmmrt
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# deep-neural-network-retention-time-prediction

## Summary

Train a heavily regularized deep neural network (cosine annealing warm restarts + stochastic weight averaging) on alvaDesc-derived fingerprints/descriptors from the 80,038-molecule METLIN SMRT dataset to predict reversed-phase LC-MS retention times, evaluated by nested stratified cross-validation.

## When to use

Use when you need to predict LC-MS retention times (RTs) for small molecules from structural features (fingerprints/descriptors), including both retained and non-retained molecules (RT < 300 s), and want state-of-the-art accuracy.

## When NOT to use

- You only have feature tables already computed and just need a simple baseline — gradient boosting models (XGBoost, LightGBM, CatBoost) are cheaper alternatives benchmarked in the same paper
- Your goal is projecting RTs between chromatographic methods from a few identified molecules — that requires the Bayesian meta-learned GP projection skill, not RT prediction
- alvaDesc license is unavailable and RDKit fingerprints are used — expect different feature sets (notebook train_with_rdkit.ipynb exists but does not reproduce the exact benchmark)

## Inputs

- SMILES / InChI / SDF / mol / mol2 / hin structure files or CSV with pubchem id + inchi/SMILES for 80,038 SMRT molecules
- alvaDesc-generated fingerprints (ECFP, MACCSFP, PFP; 2,214 bits) and molecular descriptors (5,666 features)
- Experimental retention times from the METLIN SMRT dataset (reversed-phase LC-MS)

## Outputs

- Trained DNN RT predictor (plus preprocessor) saved to a models folder
- Predicted retention times (seconds) per molecule
- Cross-validation results table with MAE and MEDAE (mean ± standard error), e.g. results/rt/rt_cv.csv
- Optuna hyperparameter-search database (e.g. results/optuna/train.db)

## How to apply

Load the 80,038 SMRT RT records with alvaDesc-derived features (5,666 descriptors and 2,214 MACCS166/ECFP/Path fingerprints). Preprocess: standardize descriptors, median-impute with missing indicators, drop zero-variance features and features correlated >0.9, remove low-variance fingerprint bits, and apply a quantile transformation to RT targets before fitting. Build the DNN regressor (3 hidden layers with large dropout rates) trained with cosine annealing warm restarts followed by Stochastic Weight Averaging; tune hidden sizes, dropout rates, activations, and annealing lengths with a TPE Bayesian search (50 iterations, maximizing negative MEDAE). Evaluate with nested stratified CV (5 outer, 7 inner folds, RT stratified into 6 bins) and report MAE and MEDAE (mean ± SE) for all, retained, and non-retained molecules.

## Related tools

- **alvaDesc** (Generates the 5,666 molecular descriptors and 2,214 ECFP/MACCSFP/PFP fingerprints used as DNN input features (commercial license required)) — https://www.alvascience.com/alvadesc/
- **cmmrt** (Python package implementing DNN training, hyperparameter search, and nested-CV evaluation of RT predictors) — https://github.com/constantino-garcia/cmmrt
- **RDKit** (Alternative fingerprint source for training the DNN without alvaDesc (see train_with_rdkit.ipynb))
- **Python** (Implementation language; install via `make install` and run experiments through the Makefile) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make install && make test_predictor  # full run: remove --smoke_test from the train_predictor/test_predictor Makefile rules
```

## Evaluation signals

- Nested-CV performance on all SMRT molecules should match the fingerprint-DNN benchmark: MAE ≈ 39.2 ± 1.2 s and MEDAE ≈ 17.2 ± 0.9 s
- Evaluation must include both retained and non-retained molecules (non-retained defined as RT < 300 s / 5 min), with per-group MAE/MEDAE reported separately
- The Makefile smoke test `make test_predictor` should produce results/rt/rt_cv.csv and an Optuna DB under results/optuna/
- Preprocessing invariants: zero-variance and correlation>0.9 features removed, quantile transformation applied to RTs before fitting
- Full (non-smoke) training requires removing the --smoke_test flag from the Makefile rule; smoke-test results are on subsampled data and will be worse than the benchmark

## Limitations

- alvaDesc is commercial software; fingerprints cannot be regenerated without a license (RDKit fingerprints are a partial alternative)
- The Makefile defaults run smoke tests (subset of regressors and data); full reproduction requires editing the Makefile and can take hours
- The repository is under active development for CEU Mass Mediator integration — the `paper` branch is the reference for reproducing paper results
- Reusing hyperparameter-search databases across runs is discouraged unless you understand partial-run reuse

## Evidence

- [abstract] The best results were obtained by a heavily regularized deep neural network trained with cosine annealing warm restarts and stochastic weight averaging, achieving a mean and median absolute errors: "The best results were obtained by a heavily regularized deep neural network trained with cosine annealing warm restarts and stochastic weight averaging"
- [methods] quantile transformation was applied to RTs before fitting.: "quantile transformation was applied to RTs before fitting."
- [methods] Highly correlated features were also eliminated (correlation >0.9): [evidence span withheld — anchor status: not_found]
- [methods] trained several state-of-the-art machine learning regressors to predict small molecules Retention Time (RT) using the 80,038 experimental RTs from the SMRT dataset.: "trained several state-of-the-art machine learning regressors to predict small molecules Retention Time (RT) using the 80,038 experimental RTs from the SMRT dataset"
- [readme] train a DNN with the methods of the paper on SMRT using RDKit fingerprints.: "train a DNN with the methods of the paper on SMRT using RDKit fingerprints."
- [readme] branch `paper` should be used as reference for reproducing the results of the paper.: "branch `paper` should be used as reference 
for reproducing the results of the paper."
