---
name: svr-hyperparameter-grid-search
description: Use when fitting a linear-kernel SVR regression model (e.g., CCS prediction
  from molecular descriptors) and the regularization parameter C and epsilon must
  be tuned by k-fold cross-validation to minimize RMSECV.
when_to_use_negative:
- The task requires nonlinear kernels (RBF, polynomial) or regularization strengths
  above 0.25 — the CCSP 2.0 grid was validated only for linear-kernel SVR with C in
  0.00390625–0.25
- Input is raw, unnormalized molecular descriptors — the grid search must operate
  on Z-transformed descriptors computed with training-set statistics, and test datapoints
  exceeding 1000 standard deviations from their mean are excluded
- The goal is classification or direct candidate filtering rather than building a
  regression model whose hyperparameters must be cross-validated before prediction
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SciKit-Learn
  role: Provides the GridSearchCV module (5-fold) and the linear-kernel SVR estimator
    whose C (0.00390625–0.25) and epsilon (0.5, 1, 5) parameters are tuned; requires
    V1.0.2 or later
- name: CCSP 2.0
  role: Jupyter notebook that encapsulates the SVR hyperparameter grid search as steps
    in the CCS prediction pipeline, re-running tuning after RFECV to rebuild final
    models
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: Execution environment for CCSP 2.0, available as a Google Colaboratory notebook
    for small-scale runs (<10,000 molecules) or a locally hosted Jupyter Lab version
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:54778e927a45ba47b16ddebc1b0b4511675a57da533a906b5171a53d59e945d1
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
  - Numpy
  - Pandas
  - PubChemPy
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# svr-hyperparameter-grid-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tunes linear-kernel support vector regression (SVR) models for CCS prediction via exhaustive 5-fold cross-validated grid search over regularization (C) and epsilon, selecting the pair that minimizes RMSECV. In CCSP 2.0 this tuning is applied both before and after recursive feature elimination (RFECV) to rebuild final per-adduct models.

## When to use

Use when fitting a linear-kernel SVR regression model (e.g., CCS prediction from molecular descriptors) and the regularization parameter C and epsilon must be tuned by k-fold cross-validation to minimize RMSECV.

## When NOT to use

- The task requires nonlinear kernels (RBF, polynomial) or regularization strengths above 0.25 — the CCSP 2.0 grid was validated only for linear-kernel SVR with C in 0.00390625–0.25
- Input is raw, unnormalized molecular descriptors — the grid search must operate on Z-transformed descriptors computed with training-set statistics, and test datapoints exceeding 1000 standard deviations from their mean are excluded
- The goal is classification or direct candidate filtering rather than building a regression model whose hyperparameters must be cross-validated before prediction

## Inputs

- Calibration-set descriptor matrix (Z-transformed Mordred 2D descriptors using training-set statistics)
- Calibration-set CCS target values (Å²) from the McLean Unified CCS Compendium adduct datasets (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions)
- RFE-selected descriptor subsets (post-RFECV) for the second tuning pass

## Outputs

- Optimal (C, epsilon) hyperparameter pair per adduct/model, selected by minimum 5-fold RMSECV
- Trained linear-kernel SVR model file per adduct
- Grid-search CV error (RMSECV) for all 21 candidate (C, epsilon) combinations

## How to apply

First split molecules ~70:30 into calibration and external validation sets and Z-transform descriptors using training-set mean and standard deviation only. Run scikit-learn's GridSearchCV over a linear-kernel SVR with seven C options spanning 0.00390625 to 0.25 (powers of two from 2^-8 to 2^-2) and three epsilon options (0.5, 1, 5) using 5-fold cross-validation, selecting the (C, epsilon) pair that minimizes RMSECV. Apply this tuning to the full 1613-descriptor Mordred feature space first, then re-run the identical grid search after RFECV removes the five lowest-ranked descriptors per iteration (stopping when RMSECV increases) to rebuild the final model on only the RFE-selected descriptor subsets (~370, 303, 414 descriptors for [M-H]-, [M+H]+, [M+Na]+). The tuned hyperparameters are reused for the final per-adduct trained model files. The repository requires scikit-learn V1.0.2 or later.

## Related tools

- **SciKit-Learn** (Provides the GridSearchCV module (5-fold) and the linear-kernel SVR estimator whose C (0.00390625–0.25) and epsilon (0.5, 1, 5) parameters are tuned; requires V1.0.2 or later)
- **CCSP 2.0** (Jupyter notebook that encapsulates the SVR hyperparameter grid search as steps in the CCS prediction pipeline, re-running tuning after RFECV to rebuild final models) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (Execution environment for CCSP 2.0, available as a Google Colaboratory notebook for small-scale runs (<10,000 molecules) or a locally hosted Jupyter Lab version) — https://github.com/facundof2016/CCSP2.0

## Examples

```
from sklearn.model_selection import GridSearchCV; from sklearn.svm import SVR; GridSearchCV(SVR(kernel='linear'), {'C':[0.00390625,0.0078125,0.015625,0.03125,0.0625,0.125,0.25],'epsilon':[0.5,1,5]}, cv=5)
```

## Evaluation signals

- The grid must cover exactly 21 candidate pairs (7 C values × 3 epsilon values) evaluated with 5-fold cross-validation under a linear kernel
- The selected (C, epsilon) pair must correspond to the minimum RMSECV, and the identical grid search must be re-run after RFECV descriptor removal to rebuild the final model
- Per-adduct validation RMSEV should fall in the reported range (~4.998, 5.127, 6.612 Å² for [M-H]-, [M+H]+, [M+Na]+)
- Median relative error on the 30% hold-out set should be at or below ~2% (reported 1.25%, 1.73%, 1.87%)
- All descriptor transformation statistics used in tuning must come from the training set only (no leakage from the 30% external validation split)

## Limitations

- The published grid is restricted to a linear kernel and a narrow C range (maximum 0.25); other kernels or stronger regularization were not validated
- Current training sets support only the tested adducts ([M-H]-, [M+H]+, [M+Na]+) and not dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments
- The specific training set must be defined prior to prediction, and running CCSP 2.0 requires familiarity with Python and scikit-learn V1.0.2 or later
- The Google Colab notebook stops after 12 hours of continuous operation and disconnects after ~20 minutes if idle, limiting long tuning runs

## Evidence

- [full_text] Tuning methodology and grid definition: [evidence span withheld — anchor status: not_found]
- [readme] Package version requirement for reproducing the grid search: [evidence span withheld — anchor status: external_doc]
- [readme] Tool purpose and user-curated training sets: [evidence span withheld — anchor status: external_doc]
