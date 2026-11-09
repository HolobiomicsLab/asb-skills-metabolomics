---
name: retention-time-regression-training
description: Use when you have a large set of molecules with experimental retention
  times (e.g., SMRT reversed-phase LC-MS data) plus alvaDesc fingerprints/descriptors
  and need to train or compare RT prediction regressors.
when_to_use_negative:
- Your molecules lack a way to generate alvaDesc features (no license); the README
  suggests RDKit fingerprints via train_with_rdkit.ipynb instead, with possibly different
  performance.
- Your goal is projecting predicted RTs to a specific chromatographic method or annotating
  metabolites from m/z and RT — use the meta-learned GP projection workflow, not RT
  regressor training.
- Your dataset is small (e.g., a handful of identified molecules); the meta-learning
  projection approach was designed to learn from as few as 10 molecules, whereas this
  skill requires tens of thousands of RTs.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: alvaDesc
  role: Generates the 5,666 molecular descriptors and 2,214 fingerprints (MACCS166,
    ECFP, PFP) used as regressor features
  repo: https://www.alvascience.com/alvadesc/
- name: cmmrt
  role: Python package implementing the RT prediction pipeline, Makefile smoke tests,
    and train/test scripts (train_model.py)
  repo: https://github.com/constantino-garcia/cmmrt
- name: XGBoost
  role: Gradient boosting regressor trained with Bayesian TPE search; also used as
    the non-retained molecule classifier (RT < 5 min) maximizing F1
- name: LightGBM
  role: GBM alternative tuned stepwise for speed during hyperparameter search
- name: CatBoost
  role: Regressor tuned with defaults and a weighted variant where non-retained molecule
    weights were varied from 1e-6 to 80
- name: RDKit
  role: Alternative fingerprint source for training a DNN on SMRT (train_with_rdkit.ipynb
    notebook)
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:cf45b873c5c7026d93cc5549ee535b68af0f0dbc9302b0e920254491da10147d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CatBoost
  - LightGBM
  - Python
  - RDKit
  - XGBoost
  - alvaDesc
  - cmmrt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# retention-time-regression-training

## Summary

Train and benchmark machine learning regressors (XGBoost, LightGBM, CatBoost, DKL, and heavily regularized DNNs) to predict chromatographic retention times from molecular fingerprints and descriptors on the 80,038-molecule METLIN SMRT dataset, using nested cross-validation with Bayesian hyperparameter search.

## When to use

Use when you have a large set of molecules with experimental retention times (e.g., SMRT reversed-phase LC-MS data) plus alvaDesc fingerprints/descriptors and need to train or compare RT prediction regressors.

## When NOT to use

- Your molecules lack a way to generate alvaDesc features (no license); the README suggests RDKit fingerprints via train_with_rdkit.ipynb instead, with possibly different performance.
- Your goal is projecting predicted RTs to a specific chromatographic method or annotating metabolites from m/z and RT — use the meta-learned GP projection workflow, not RT regressor training.
- Your dataset is small (e.g., a handful of identified molecules); the meta-learning projection approach was designed to learn from as few as 10 molecules, whereas this skill requires tens of thousands of RTs.

## Inputs

- SMRT dataset (80,038 experimental reversed-phase LC-MS retention times) as CSV with compound identifiers (pubchem id, inchi or SMILES)
- alvaDesc-generated feature files: ECFP, MACCSFP, PFP fingerprints (2,214 bits) and molecular descriptors (5,666), sequentially concatenated strings
- cmmrt package installed with Python dependencies

## Outputs

- rt_cv.csv summary of MAE and MEDAE across nested CV folds, grouped by regressor and feature set
- Trained blender and base-regressor models saved under saved_models/
- Optuna hyperparameter search databases (results/optuna/train.db, results/optuna/cv.db)

## How to apply

Load the SMRT dataset and build feature sets with alvaDesc: 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, ECFP, and Path Fingerprints), alone and combined, then standardize descriptors, apply median imputation with a missing indicator, remove zero-variance features, and eliminate features correlated above 0.9. Augment features with a predicted non-retained binary label by training an XGBoost classifier (RT < 5 minutes = 300 s) on fingerprint features. Train XGBoost, LightGBM, CatBoost, DKL, and DNN regressors with 50 iterations of Bayesian TPE hyperparameter search maximizing negative MEDAE; for DNNs apply quantile transformation of RTs plus cosine annealing warm restarts with stochastic weight averaging. Run nested stratified cross-validation (5 outer, 7 inner folds, RTs binned into 6 strata) to avoid data leakage, then build a blending ensemble by holding out 20% of training data and fitting a Bayesian-tuned random-forest meta-regressor. Aggregate MAE and MEDAE per fold into a summary CSV (rt_cv.csv) grouped by regressor and feature set; the best configuration is expected to be the regularized DNN on fingerprints with ~39.2 ± 1.2 s MAE and ~17.2 ± 0.9 s MEDAE.

## Related tools

- **alvaDesc** (Generates the 5,666 molecular descriptors and 2,214 fingerprints (MACCS166, ECFP, PFP) used as regressor features) — https://www.alvascience.com/alvadesc/
- **cmmrt** (Python package implementing the RT prediction pipeline, Makefile smoke tests, and train/test scripts (train_model.py)) — https://github.com/constantino-garcia/cmmrt
- **XGBoost** (Gradient boosting regressor trained with Bayesian TPE search; also used as the non-retained molecule classifier (RT < 5 min) maximizing F1)
- **LightGBM** (GBM alternative tuned stepwise for speed during hyperparameter search)
- **CatBoost** (Regressor tuned with defaults and a weighted variant where non-retained molecule weights were varied from 1e-6 to 80)
- **RDKit** (Alternative fingerprint source for training a DNN on SMRT (train_with_rdkit.ipynb notebook)) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make test_predictor  # then, for full training: python cmmrt/rt/train_model.py --storage sqlite:///results/optuna/train.db --save_to saved_models
```

## Evaluation signals

- rt_cv.csv is produced with per-fold MAE and MEDAE for all-molecule, non-retained, and retained subsets, grouped by regressor family and feature set
- Best model MAE should fall near 39.2 ± 1.2 s and MEDAE near 17.2 ± 0.9 s on the full SMRT dataset; smoke tests will differ but the run must complete without errors
- Fingerprint-only feature sets should outperform descriptor-only sets, per the paper's finding that 'fingerprints tend to perform better'
- Nested CV structure intact: no leakage — inner 7-fold Bayesian search (50 trials) nested within 5 stratified outer folds with RTs binned into 6 strata
- The saved blender in saved_models/ loads and produces RT predictions for new SMILES after alvaDesc fingerprint generation via generate_vector_fingerprints()

## Limitations

- alvaDesc is commercial/under license; some descriptors may not be computable, requiring the median imputation with missing indicator path
- The Makefile smoke tests train only a subset of regressors on a subsampled SMRT data for a few epochs; complete results require removing --smoke_test and waiting hours
- Only the `paper` branch should be used as reference for reproducing the paper's results, as the repo is being developed for CEU Mass Mediator integration
- Hyparameter search databases should not be reused across runs unless partial-run reuse is understood; a fresh SQLite storage is recommended

## Evidence

- [abstract] The best results were obtained by a heavily regularized DNN trained with cosine annealing warm restarts and stochastic weight averaging, achieving a mean and median absolute errors of 39.2±1.2 s and 17.2 ± 0.9 s: "The best results were obtained by a heavily regularized DNN trained with cosine annealing warm restarts and stochastic weight averaging"
- [methods] feature engineering thresholds: [evidence span withheld — anchor status: not_found]
- [methods] non-retained label definition: "a molecule was considered non-retained if its Retention Time (RT) was smaller than 5 minutes."
- [methods] feature generation provenance: "Descriptors and fingerprints were generated with the alvaDesc software."
- [readme] nested CV procedure: "Tests the performance of a subset of regressors on a subset of the SMRT database using nested cross-validation. A summary of the results is stored in the CSV `results/rt/rt_cv.csv`."
- [readme] reproducibility branch: "branch `paper` should be used as reference for reproducing the results of the paper."
