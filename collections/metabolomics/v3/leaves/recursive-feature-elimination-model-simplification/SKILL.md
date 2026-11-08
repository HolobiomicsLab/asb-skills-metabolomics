---
name: recursive-feature-elimination-model-simplification
description: Use when a high-dimensional descriptor matrix (e.g., 1613 Mordred descriptors)
  must be reduced to a minimal predictive feature subset for regression, with the
  stopping point determined by cross-validated RMSE rather than a fixed feature count.
when_to_use_negative:
- When the feature space is already small or model interpretability is not a goal
  — the elimination loop adds compute without benefit
- When the target task is classification or filtering (e.g., ΔCCS threshold filtering
  at 2.8%) rather than continuous regression with an SVR base estimator
- When descriptors have not yet been scrubbed — constant descriptors across calibration
  molecules and inapplicable descriptors must be removed before Z-transformation and
  RFECV
edam_operation: http://edamontology.org/operation_3912
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3524
tools:
- name: SciKit-Learn
  role: 'Provides the linear-kernel SVR base estimator, GridSearchCV hyperparameter
    tuning (C: 0.00390625–0.25; epsilon: 0.5, 1, 5; 5-fold CV), and the RFECV elimination
    loop'
- name: Mordred V1.2.0
  role: Generates the 1613 2D molecular descriptors that are the input space being
    reduced by RFECV
- name: RDKit
  role: Encodes SMILES-derived MOL files from which the Mordred descriptors are calculated
- name: CCSP 2.0
  role: Open-source Jupyter Notebook implementing the full descriptor calculation,
    split generation, and RFECV-SVR training workflow
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:2d7e9d31b1ab1b96c2a3d06849b7a948362d1af2b482bdd24407cdaf70b15b24
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - Mordred V1.2.0
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: http://edamontology.org/operation_3912
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# Recursive Feature Elimination Model Simplification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Iteratively removes the lowest-ranked molecular descriptors via recursive feature elimination with cross-validation (RFECV) to simplify SVR models — as used in CCSP 2.0 to reduce 1613 Mordred 2D descriptors down to a compact, high-accuracy CCS prediction feature set.

## When to use

Use when a high-dimensional descriptor matrix (e.g., 1613 Mordred descriptors) must be reduced to a minimal predictive feature subset for regression, with the stopping point determined by cross-validated RMSE rather than a fixed feature count.

## When NOT to use

- When the feature space is already small or model interpretability is not a goal — the elimination loop adds compute without benefit
- When the target task is classification or filtering (e.g., ΔCCS threshold filtering at 2.8%) rather than continuous regression with an SVR base estimator
- When descriptors have not yet been scrubbed — constant descriptors across calibration molecules and inapplicable descriptors must be removed before Z-transformation and RFECV

## Inputs

- Descriptor matrix of 1613 2D Mordred (V1.2.0) descriptors calculated from RDKit-encoded MOL files, with constant and inapplicable descriptors already discarded
- Z-transformed descriptor values using training-set statistics (excluding test points >1000 standard deviations from the mean)
- Calibration/validation split assignments (70%/30% per iteration) for the adduct dataset being modeled ([M-H]-, [M+H]+, or [M+Na]+)

## Outputs

- Reduced descriptor feature set per iteration (descriptors surviving RFECV elimination)
- Per-iteration RMSECV trajectory tracking when elimination stops
- Trained simplified linear SVR model per split yielding RMSEV and MRE on the validation set

## How to apply

After descriptor calculation and pruning of constants/inapplicable columns, Z-transform remaining descriptors using training-set statistics, then for each calibration/validation split (e.g., 101 iterations of 70/30 assignment) train the base regressor — in CCSP 2.0 a linear-kernel SVR with L2 regularization, with C tuned over seven grid values (0.00390625–0.25) and epsilon over (0.5, 1, 5) via 5-fold GridSearchCV. Run RFECV removing the five lowest-ranked descriptors per iteration and continue eliminating until the cross-validated RMSE (RMSECV) increases, which defines the stopping point. Rationale: dropping weakly informative descriptors one small batch at a time keeps the model compact and interpretable while RMSECV guards against removing features that contribute predictive signal. Record the surviving descriptor set and RMSECV per split so the median-RMSEV model can be inspected and reproduced.

## Related tools

- **SciKit-Learn** (Provides the linear-kernel SVR base estimator, GridSearchCV hyperparameter tuning (C: 0.00390625–0.25; epsilon: 0.5, 1, 5; 5-fold CV), and the RFECV elimination loop)
- **Mordred V1.2.0** (Generates the 1613 2D molecular descriptors that are the input space being reduced by RFECV)
- **RDKit** (Encodes SMILES-derived MOL files from which the Mordred descriptors are calculated)
- **CCSP 2.0** (Open-source Jupyter Notebook implementing the full descriptor calculation, split generation, and RFECV-SVR training workflow) — https://github.com/facundof2016/CCSP2.0

## Examples

```
from sklearn.feature_selection import RFECV; from sklearn.svm import SVR; rfecv = RFECV(SVR(kernel='linear'), step=5, cv=5, scoring='neg_root_mean_squared_error'); rfecv.fit(X_train, y_train)
```

## Evaluation signals

- RMSECV should decrease monotonically during elimination and the loop must stop at the first iteration where RMSECV increases
- Reproduced median-model metrics should match reported values: median MRE of 1.251%/1.729%/1.874% and RMSEV of 4.998/5.127/6.612 Å² for [M-H]-/[M+H]+/[M+Na]+ test sets
- 70–78% of validation predictions within 3% error and 90–92% within 5% error per adduct type ([M-H]-: 78%/92%; [M+H]+: 70%/92%; [M+Na]+: 74%/90%)
- All per-split and median-model metrics for the three adducts are written to ccsp2_validation_metrics.csv
- No constant or inapplicable descriptors remain in the feature set fed to RFECV, and Z-transformation uses training-set statistics only (no test-set leakage)

## Limitations

- RFECV must be rerun per split — the descriptor set is selected independently within each of the 101 calibration iterations, so no single canonical feature list is produced
- The base SVR must be scikit-learn V1.0.2 or later per the README requirements
- Test points >1000 standard deviations from the training mean are excluded from Z-transformation; extreme molecular structures outside the calibration domain may degrade prediction quality
- Training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments

## Evidence

- [other] RFECV stopping rule and base estimator: [evidence span withheld — anchor status: not_found]
- [readme] Sklearn version requirement: [evidence span withheld — anchor status: external_doc]
