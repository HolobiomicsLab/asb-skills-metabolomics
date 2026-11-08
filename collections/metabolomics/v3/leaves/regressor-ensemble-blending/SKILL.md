---
name: regressor-ensemble-blending
description: Use when several base regressors have been trained on molecular fingerprints/descriptors
  for RT prediction and you need a stacked 'blender' that combines their predictions
  to minimize median absolute error under nested CV.
when_to_use_negative:
- Only a single base regressor is being trained and no comparison or combination of
  regressors is intended
- Input is already a matrix of model predictions rather than SMILES/SDF structures
  from which alvaDesc fingerprints and descriptors must be generated
- The task is RT projection between chromatographic methods (meta-learned GP), not
  regressor combination
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2258
tools:
- name: cmmrt
  role: Package providing train_model.py and Makefile rules (train_predictor/test_predictor)
    that train the blender and base regressors with Bayesian hyperparameter search
    and nested CV
  repo: https://github.com/constantino-garcia/cmmrt
- name: XGBoost
  role: Base GBM regressor whose predictions feed the blending ensemble; also used
    for the non-retained classification feature
- name: LightGBM
  role: Base regressor alternative to XGBoost, tuned stepwise for speed, included
    in the blender
- name: CatBoost
  role: Weighted base regressor (non-retained weights 1e-6 to 80, ratio 1/40) whose
    predictions are stacked into the blender
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:66a3cba6072ff83b79774bf323bc3a58dbcd4c6fb262c9ba6b425ae7e37f2b6d
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# regressor-ensemble-blending

## Summary

Stacks multiple RT-prediction regressors (XGBoost, LightGBM, CatBoost, DNN) trained on the 80,038-molecule SMRT dataset into a blending ensemble by training a Bayesian-tuned random-forest meta-regressor on held-out base-model predictions, to outperform any single regressor on MEDAE/MAE.

## When to use

Use when several base regressors have been trained on molecular fingerprints/descriptors for RT prediction and you need a stacked 'blender' that combines their predictions to minimize median absolute error under nested CV.

## When NOT to use

- Only a single base regressor is being trained and no comparison or combination of regressors is intended
- Input is already a matrix of model predictions rather than SMILES/SDF structures from which alvaDesc fingerprints and descriptors must be generated
- The task is RT projection between chromatographic methods (meta-learned GP), not regressor combination

## Inputs

- SMRT dataset (80,038 experimental retention times) with alvaDesc feature sets: 2,214 ECFP/MACCSFP/PFP fingerprints, 5,666 descriptors, and fingerprints+descriptors combined
- Predictions of trained base regressors (XGBoost, LightGBM, CatBoost, weighted CatBoost, DKL, DNN) on the 20% held-out blending split
- Trained non-retained classifier feature (XGBoost, RT < 5 min) appended to the feature matrix

## Outputs

- Trained blending ensemble (random-forest meta-regressor containing all base regressors) saved to the saved_models folder
- Hyperparameter search log in SQLite DB results/optuna/train.db
- rt_cv.csv summarizing MAE and MEDAE across folds for all molecules, non-retained, and retained subsets, grouped by regressor family and feature set

## How to apply

Hold out 20% of the training set (controlled via --train_size) and store each base regressor's predictions on it; the remainder trains the base models. Train a random-forest meta-regressor on those stored predictions, tuning it with Bayesian TPE search (Optuna, results in sqlite:///results/optuna/train.db) maximizing negative MEDAE. Wrap the blender so it contains all base regressors, then evaluate inside nested stratified CV (5 outer, 7 inner folds, RTs stratified into 6 bins) to avoid leakage, minimizing MEDAE on validation splits. Aggregate MAE and MEDAE across folds for all molecules plus non-retained (RT < 300 s) and retained subsets into a grouped comparison CSV (rt_cv.csv) by regressor family and feature set (fingerprints, descriptors, fingerprints+descriptors).

## Related tools

- **cmmrt** (Package providing train_model.py and Makefile rules (train_predictor/test_predictor) that train the blender and base regressors with Bayesian hyperparameter search and nested CV) — https://github.com/constantino-garcia/cmmrt
- **XGBoost** (Base GBM regressor whose predictions feed the blending ensemble; also used for the non-retained classification feature)
- **LightGBM** (Base regressor alternative to XGBoost, tuned stepwise for speed, included in the blender)
- **CatBoost** (Weighted base regressor (non-retained weights 1e-6 to 80, ratio 1/40) whose predictions are stacked into the blender)

## Examples

```
python cmmrt/rt/train_model.py --storage sqlite:///results/optuna/train.db --save_to saved_models --train_size 0.8 --trials 50
```

## Evaluation signals

- The best resulting blender (which contains all other regressors) is saved to the saved_models folder after train_predictor completes
- results/rt/rt_cv.csv exists and contains per-fold MAE/MEDAE rows grouped by regressor family and feature set (fps, descs, fps+descs)
- Blender MEDAE on the outer CV folds is at or below the best single base regressor's MEDAE (target ballpark: median absolute error 17.2 ± 0.9 s for the DNN)
- Hyperparameter trials are recorded in results/optuna/train.db (training) and results/optuna/cv.db (nested within the outer loop)
- The 80/20 base/meta split is respected: train_size parameter set so the remainder trains the meta-regressor

## Limitations

- Makefile rules run with --smoke_test on a subsample of SMRT and reduced models/trials; the FIXME line must be removed for full training with all regressors
- README advises not reusing hyperparameter SQLite databases unless you understand how to reuse partial runs
- Fingerprint/descriptor generation requires alvaDesc under license; full nested CV with Bayesian search over all regressors takes hours

## Evidence

- [readme] The best resulting blender (which contains all other regressors) is saved to the `saved_models` folder.: "The best resulting blender (which contains all other regressors) is saved to the `saved_models` folder."
- [readme] train_size param description: "Percentage of the training set to train the base classifiers. The remainder is used to train the meta-classifier"
- [readme] results summary location: "A summary of the results is stored in the CSV `results/rt/rt_cv.csv`. A summary of all tested models during hyperparameter search is stored in `results/optuna/cv.db`"
- [intro] regressor comparison scope: "In this paper we have tested the performance of several state-of-the-art machine learning models for the task of Retention Time (RT) prediction using the SMRT dataset."
- [methods] CatBoost weighting rationale: "CatBoost [36]: an interesting question regarding the non-retained molecules is if their inclusion as training data improves the performance of the regressors."
