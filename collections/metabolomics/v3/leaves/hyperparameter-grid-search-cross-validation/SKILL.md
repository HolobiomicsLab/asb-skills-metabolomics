---
name: hyperparameter-grid-search-cross-validation
description: Use when training SVR (or similar) regression models whose accuracy depends
  on hyperparameters (e.g. regularization strength) and you need an unbiased performance
  estimate via repeated random train/test splits with a grid search over parameters.
when_to_use_negative:
- The model needs no tuning (e.g. a fixed, already-validated predictor) and you only
  want point CCS predictions on new molecules rather than training/validation
- The dataset is too small to support repeated train/test splits — CCSP 2.0 relies
  on 138–170 labeled compounds per adduct type
- You need to predict adduct types absent from the training set, such as dimers ([2M+H]+,
  [2M−H]−) or in-source fragments, which the current CCSP 2.0 training sets do not
  cover
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook that implements the GridSearchCV-tuned linear-kernel
    SVR training and the 101-split validation protocol for CCS prediction
  repo: https://github.com/facundof2016/CCSP2.0
- name: Jupyter Notebook
  role: Execution environment (Python 3, e.g. via Anaconda) in which the CCSP 2.0
    .ipynb notebook runs the grid search and cross-validation; requires scikit-learn
    V1.0.2 or later
- name: Mordred
  role: Python package supplying the 1613 molecular descriptors that form the feature
    space reduced by RFE before SVR tuning
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.2c03491
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:a7114fab766b8bbd3c8186ba7d9c9d9215423ef79c114939df52e94d24488cb5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hyperparameter-grid-search-cross-validation

## Summary

Tunes and validates regression models by exhaustively searching hyperparameter combinations with GridSearchCV across multiple random train/test splits, as used in CCSP 2.0 where linear-kernel SVR models are tuned over 101 randomly selected splits per adduct type.

## When to use

Use when training SVR (or similar) regression models whose accuracy depends on hyperparameters (e.g. regularization strength) and you need an unbiased performance estimate via repeated random train/test splits with a grid search over parameters.

## When NOT to use

- The model needs no tuning (e.g. a fixed, already-validated predictor) and you only want point CCS predictions on new molecules rather than training/validation
- The dataset is too small to support repeated train/test splits — CCSP 2.0 relies on 138–170 labeled compounds per adduct type
- You need to predict adduct types absent from the training set, such as dimers ([2M+H]+, [2M−H]−) or in-source fragments, which the current CCSP 2.0 training sets do not cover

## Inputs

- Labeled training data: CCS measurements from the McLean CCS Compendium partitioned by adduct type ([M−H]−, [M+H]+, [M+Na]+)
- Feature matrix of molecular descriptors (1613-entry Mordred descriptor library, reduced via recursive feature elimination)
- Hyperparameter grid specification for the SVR model (e.g. linear-kernel regularization parameters)

## Outputs

- Trained linear-kernel SVR models per adduct type and per train/test split (101 splits per adduct)
- Per-adduct median relative error (MRE) and RMSEV metrics table (e.g. written to ccs_model_performance.csv)
- Best hyperparameter settings selected by GridSearchCV for each split

## How to apply

Partition the labeled dataset (here, CCS values from the McLean CCS Compendium split by adduct type: 170 [M−H]−, 155 [M+H]+, 138 [M+Na]+ compounds) and reduce the feature space first (e.g. recursive feature elimination over the 1613-entry Mordred descriptor library). Generate 101 randomly selected train/test splits and keep the test sets consistent across all models being compared, so algorithm benchmarks are fair. For each split, run GridSearchCV to tune a linear-kernel SVR, then predict CCS values on the held-out test set and record the median relative error (MRE) and root-mean-square error of validation (RMSEV) in Å². In CCSP 2.0 this protocol yielded median MREs of 1.251% (RMSEV 4.998 Å²) for [M−H]−, 1.729% (5.127 Å²) for [M+H]+, and 1.874% (6.612 Å²) for [M+Na]+; report metrics per adduct type rather than pooled.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook that implements the GridSearchCV-tuned linear-kernel SVR training and the 101-split validation protocol for CCS prediction) — https://github.com/facundof2016/CCSP2.0
- **Jupyter Notebook** (Execution environment (Python 3, e.g. via Anaconda) in which the CCSP 2.0 .ipynb notebook runs the grid search and cross-validation; requires scikit-learn V1.0.2 or later)
- **Mordred** (Python package supplying the 1613 molecular descriptors that form the feature space reduced by RFE before SVR tuning)

## Examples

```
from sklearn.svm import SVR; from sklearn.model_selection import GridSearchCV; gs = GridSearchCV(SVR(kernel='linear'), param_grid={'C':[0.1,1,10]}, cv=5).fit(X_train, y_train)
```

## Evaluation signals

- Median relative error per adduct reproduces the reported values: 1.251% ([M−H]−), 1.729% ([M+H]+), 1.874% ([M+Na]+) within reasonable tolerance
- RMSEV per adduct falls near the reported 4.998 Å² ([M−H]−), 5.127 Å² ([M+H]+), and 6.612 Å² ([M+Na]+)
- The same 101 randomly selected test sets are reused consistently across all compared models/algorithms
- Training set sizes match the McLean CCS Compendium partitions: 170 [M−H]−, 155 [M+H]+, and 138 [M+Na]+ compounds
- Metrics table (e.g. ccs_model_performance.csv) contains MRE and RMSEV per adduct type rather than pooled across adducts

## Limitations

- Requires scikit-learn V1.0.2 or later per the repository README
- The training set must be defined prior to predicting CCS values — a stated drawback of these ML methods
- Current training sets cover only common adducts ([M−H]−, [M+H]+, [M+Na]+, [M+K]+) and cannot predict dimers or less common adducts/in-source fragments
- In Google Colab, continuous notebook operation is limited to ~12 hours with idle disconnects after ~20 minutes, constraining long grid searches; the locally hosted Jupyter Lab version is recommended for large training sets

## Evidence

- [abstract] CCS prediction accuracy was tested using CCS values in the McLean CCS Compendium with median relative errors of 1.25, 1.73 and 1.87% for the 170 [M-H]-, 155 [M+H]+ and 138 [M+Na]+ adducts tested.: [evidence span withheld — anchor status: not_found]
