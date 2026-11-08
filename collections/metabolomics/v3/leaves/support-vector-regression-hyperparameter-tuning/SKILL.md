---
name: support-vector-regression-hyperparameter-tuning
description: Use when building SVR regression models from molecular descriptor tables
  and you need systematic hyperparameter (C, epsilon) selection via cross-validation
  plus descriptor reduction before validation.
when_to_use_negative:
- When the training set targets dimers ([2M+H]+, [2M-H]-), less common adducts, or
  in-source fragments — the article's training sets do not support these
- When a single pre-defined train/test split must be preserved (e.g., external benchmarking
  against CCSbase, DeepCCS, AllCCS) rather than repeated random 70/30 resampling
- When the user cannot work in Python/scikit-learn — the workflow requires programming
  familiarity and sklearn V1.0.2 or later
edam_operation: http://edamontology.org/operation_3949
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: CCSP 2.0
  role: Jupyter Notebook implementation wrapping the SVR tuning, RFECV descriptor
    elimination, and validation workflow
  repo: https://github.com/facundof2016/CCSP2.0
- name: SciKit-Learn
  role: Provides linear-kernel SVR, GridSearchCV 5-fold cross-validation, and RFECV
    for descriptor elimination
- name: Mordred V1.2.0
  role: Computes the 1613 2D molecular descriptors that form the SVR feature space
    before elimination
- name: RDKit
  role: Encodes MOL files for each SMILES to enable descriptor calculation
- name: Jupyter Notebook
  role: Hosting format for the workflow (Google Colab or local Jupyter Lab variants)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:8cb8b068b40f857e2b57d51babb9ed4535fda9b9159a070541762235985dab16
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
  edam_operation: http://edamontology.org/operation_3949
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# support-vector-regression-hyperparameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tunes linear-kernel support vector regression models for CCS prediction using GridSearchCV over C and epsilon, coupled with RFECV descriptor elimination, as implemented in the CCSP 2.0 Jupyter Notebook workflow.

## When to use

Use when building SVR regression models from molecular descriptor tables and you need systematic hyperparameter (C, epsilon) selection via cross-validation plus descriptor reduction before validation.

## When NOT to use

- When the training set targets dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments — the article's training sets do not support these
- When a single pre-defined train/test split must be preserved (e.g., external benchmarking against CCSbase, DeepCCS, AllCCS) rather than repeated random 70/30 resampling
- When the user cannot work in Python/scikit-learn — the workflow requires programming familiarity and sklearn V1.0.2 or later

## Inputs

- Descriptor table (molecules × Mordred 2D descriptors, e.g., 1613 features) with CCS labels per adduct set ([M-H]-, [M+H]+, [M+Na]+ from the McLean Unified CCS Compendium)
- Calibration/validation split assignments (70%/30% per iteration, 101 iterations)
- SVR hyperparameter grids: C = seven grid values 0.00390625–0.25; epsilon = {0.5, 1, 5}

## Outputs

- Trained linear-kernel SVR models with tuned C and epsilon per iteration
- Per-iteration RMSEV and MRE on the 30% validation sets
- Median-RMSEV model metrics per adduct (median MRE, RMSEV, fraction of predictions within 3% and 5% error)
- RFECV-selected descriptor subsets (five lowest-ranked descriptors removed per iteration until RMSECV increases)

## How to apply

Randomly split the descriptorized dataset (e.g., 1613 Mordred 2D descriptors computed from MOL files) into 70% calibration and 30% validation molecules. Train a linear-kernel SVR with L2 regularization and tune the regularization parameter C over a seven-value grid (0.00390625–0.25) and epsilon over (0.5, 1, 5) using 5-fold cross-validation with scikit-learn's GridSearchCV. After tuning, apply RFECV to remove the five lowest-ranked descriptors per iteration until RMSECV increases, retaining the minimal descriptor subset that minimizes cross-validated error. Repeat the entire split-train-tune-eliminate cycle over 101 iterations, computing RMSEV and MRE on the validation set of each split, and select the median-RMSEV split for reported metrics. This repeated random-split design with median-model selection guards against optimistic reporting from a single favorable train/test partition.

## Related tools

- **CCSP 2.0** (Jupyter Notebook implementation wrapping the SVR tuning, RFECV descriptor elimination, and validation workflow) — https://github.com/facundof2016/CCSP2.0
- **SciKit-Learn** (Provides linear-kernel SVR, GridSearchCV 5-fold cross-validation, and RFECV for descriptor elimination)
- **Mordred V1.2.0** (Computes the 1613 2D molecular descriptors that form the SVR feature space before elimination)
- **RDKit** (Encodes MOL files for each SMILES to enable descriptor calculation)
- **Jupyter Notebook** (Hosting format for the workflow (Google Colab or local Jupyter Lab variants))

## Examples

```
from sklearn.model_selection import GridSearchCV; gs = GridSearchCV(SVR(kernel='linear'), param_grid={'C':[0.00390625,0.0078125,0.015625,0.03125,0.0625,0.125,0.25],'epsilon':[0.5,1,5]}, cv=5, scoring='neg_root_mean_squared_error')
```

## Evaluation signals

- Median-model validation metrics match reported values: median MRE of 1.251%/1.729%/1.874% and RMSEV of 4.998/5.127/6.612 Å² for [M-H]-/[M+H]+/[M+Na]+
- 70–78% of validation predictions fall within 3% error and 90–92% within 5% error for each adduct type (78%/70%/74% within 3%; 92%/92%/90% within 5%)
- GridSearchCV converges within the specified grids: C across seven values from 0.00390625 to 0.25, epsilon among {0.5, 1, 5}, with 5-fold CV
- RFECV stops removing descriptors (five per iteration) at the point RMSECV would increase, not before
- Across 101 iterations, per-split RMSEV distribution is stable enough that the median-RMSEV split reproduces the published figures

## Limitations

- Current training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments
- The specific training set must be defined prior to CCS prediction
- Requires Sklearn V1.0.2 or later; a full list of package requirements is in the repository's requirements.txt
- Use requires familiarity with a programming language such as Python, limiting accessibility
- CCS predictions across multiple IM-MS instrument types (drift tube, travelling wave) are not yet explored

## Evidence

- [methods] SVR tuning grid and CV scheme: [evidence span withheld — anchor status: not_found]
- [methods] Descriptor elimination procedure: [evidence span withheld — anchor status: not_found]
- [methods] Repeated random splits: [evidence span withheld — anchor status: not_found]
- [readme] Package requirement: [evidence span withheld — anchor status: external_doc]
