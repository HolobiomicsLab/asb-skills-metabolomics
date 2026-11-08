---
name: gradient-boosting-regressor-tuning
description: Use when predicting continuous chromatographic retention times from molecular
  fingerprints/descriptors and you need tuned GBM regressors (XGBoost, LightGBM, CatBoost)
  compared by MAE/MEDAE under nested CV.
when_to_use_negative:
- Input structures lack a license or access to alvaDesc, so fingerprints/descriptors
  cannot be generated (RDKit fingerprints via train_with_rdkit.ipynb is a fallback,
  not equivalent)
- The task is chromatographic-method projection or metabolite annotation ranking,
  not regressor training — use the Bayesian meta-learned GP projection workflow instead
- Only a quick sanity check is needed; the Makefile smoke tests use subsampled data
  and reduced trials and will not reproduce the published errors
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: alvaDesc
  role: Generates the 5,666 descriptors and 2,214 ECFP/MACCSFP/PFP fingerprints that
    serve as regressor features
- name: XGBoost
  role: GBM regressor tuned by Bayesian TPE; also trained as a non-retained (RT <
    5 min) binary classifier maximizing F1 to add a flag feature
- name: LightGBM
  role: GBM regressor tuned stepwise to reduce search time, with optimizations for
    speed and memory usage
- name: CatBoost
  role: Weighted GBM regressor; non-retained sample weights varied from 1e-6 to 80
    (ratio 1/40) to test whether including non-retained molecules helps
- name: cmmrt
  role: Python package implementing the training/nested-CV/blending pipeline, driven
    by Makefile rules and CLI scripts with Optuna storage
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:009f7f1fd02bb20ad63eed887968ae7422b59c1d7e7766e30d828bfdb4cab602
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gradient-boosting-regressor-tuning

## Summary

Tunes and benchmarks gradient boosting regressors (XGBoost, LightGBM, CatBoost) for retention time (RT) prediction on the 80,038-molecule SMRT dataset, using Bayesian TPE hyperparameter search that minimizes median absolute error (MEDAE) and evaluating under nested stratified cross-validation.

## When to use

Use when predicting continuous chromatographic retention times from molecular fingerprints/descriptors and you need tuned GBM regressors (XGBoost, LightGBM, CatBoost) compared by MAE/MEDAE under nested CV.

## When NOT to use

- Input structures lack a license or access to alvaDesc, so fingerprints/descriptors cannot be generated (RDKit fingerprints via train_with_rdkit.ipynb is a fallback, not equivalent)
- The task is chromatographic-method projection or metabolite annotation ranking, not regressor training — use the Bayesian meta-learned GP projection workflow instead
- Only a quick sanity check is needed; the Makefile smoke tests use subsampled data and reduced trials and will not reproduce the published errors

## Inputs

- SMRT dataset of 80,038 molecules with experimental RTs and PubChem IDs/SMILES/InChI structures
- alvaDesc-generated fingerprints (ECFP, MACCSFP, PFP concatenated) and/or 5,666 molecular descriptors per molecule
- Optional SDF/SMILES structure files processed via build_data_cmm.py / build_data_smrt.py

## Outputs

- Trained regressor and blender models saved to the saved_models folder
- Comparison summary CSV (results/rt/rt_cv.csv) with MAE and MEDAE per regressor family and feature set, for all molecules plus non-retained and retained subsets
- Optuna SQLite databases (results/optuna/train.db, results/optuna/cv.db) logging all hyperparameter search trials

## How to apply

Build three feature sets with alvaDesc (2,214 ECFP/MACCSFP/PFP fingerprints, 5,666 descriptors, and both combined), applying standardization, median imputation with a missing indicator, removal of zero-variance features, and elimination of features correlated above 0.9. Optionally add a binary non-retained flag by training an XGBoost classifier (maximize F1) on molecules with RT < 300 s, and pass this flag as a feature. Tune XGBoost and weighted CatBoost (non-retained sample weights varied from 1e-6 to 80 with ratio 1/40) with 50 iterations of Bayesian TPE search maximizing negative MEDAE; tune LightGBM stepwise for speed, and run a CatBoost variant with defaults. Evaluate all regressors with nested stratified cross-validation (5 outer, 7 inner folds), stratifying RTs into 6 bins and selecting hyperparameters by MEDAE on inner validation splits to avoid leakage. Finally, build a blending ensemble by holding out 20% of training data, storing base-regressor predictions on it, and training a Bayesian-tuned random-forest meta-regressor; aggregate MAE/MEDAE across folds into a comparison summary CSV (rt_cv.csv).

## Related tools

- **alvaDesc** (Generates the 5,666 descriptors and 2,214 ECFP/MACCSFP/PFP fingerprints that serve as regressor features)
- **XGBoost** (GBM regressor tuned by Bayesian TPE; also trained as a non-retained (RT < 5 min) binary classifier maximizing F1 to add a flag feature)
- **LightGBM** (GBM regressor tuned stepwise to reduce search time, with optimizations for speed and memory usage)
- **CatBoost** (Weighted GBM regressor; non-retained sample weights varied from 1e-6 to 80 (ratio 1/40) to test whether including non-retained molecules helps)
- **cmmrt** (Python package implementing the training/nested-CV/blending pipeline, driven by Makefile rules and CLI scripts with Optuna storage) — https://github.com/constantino-garcia/cmmrt

## Examples

```
python cmmrt/rt/train_model.py --storage sqlite:///results/optuna/train.db --save_to saved_models
```

## Evaluation signals

- Nested CV summary CSV results/rt/rt_cv.csv exists and reports MAE and MEDAE per regressor grouped by family and feature set (fingerprints tended to outperform descriptors)
- XGBoost and weighted-CatBoost runs perform 50 Bayesian TPE trials maximizing negative MEDAE, logged in results/optuna/train.db
- Best tuned models should land near the paper's headline errors (e.g., best regressor MEDAE ≈ 17.2 ± 0.9 s, MAE ≈ 39.2 ± 1.2 s on SMRT), with both retained and non-retained molecules included
- Blender model (random-forest meta-regressor over held-out 20% base-model predictions) is saved to saved_models after train_model.py
- RTs stratified into 6 bins across 5 outer / 7 inner folds; no hyperparameter reuse across outer folds (fresh Optuna DB per run)

## Limitations

- README smoke tests (make train_predictor / make test_predictor) run on subsets with few epochs/trials; the --smoke_test flag must be removed for complete training, and reusing hyperparameter search databases across runs is discouraged
- Fingerprint generation requires alvaDesc under license; branch 'paper' is the reference for reproducing published results
- XGBoost and weighted CatBoost are tuned with 50 TPE iterations, while one CatBoost variant uses defaults and LightGBM is tuned stepwise, so search budgets differ across regressor families

## Evidence

- [methods] Gradient Boosting Machine (lightGBM) [35]: it is a well-known alternative to XGBoost, with optimizations for speed and memory usage: "Gradient Boosting Machine (lightGBM) [35]: it is a well-known alternative to XGBoost, with optimizations for speed and memory usage"
- [methods] CatBoost inclusion of non-retained molecules: "CatBoost [36]: an interesting question regarding the non-retained molecules is if their inclusion as training data improves the performance of the regressors."
- [methods] median imputation and correlated feature removal: [evidence span withheld — anchor status: not_found]
- [readme] README make train_predictor description: "Trains a subset of regressors on a subset of the
    SMRT database using Bayesian hyperparameter search. The best resulting blender"
- [readme] README nested CV results files: "A summary of the results is stored in the CSV `results/rt/rt_cv.csv`. A summary of all tested
    models during hyperparameter search is stored in `results/optuna/cv.db` (nested"
- [methods] alvaDesc feature generation: "5, 666 molecular descriptors and 2, 214 fingerprints (MACCS166, Extended Connectivity, and Path Fingerprints fingerprints) were generated with
 the alvaDesc software"
