---
name: recursive-feature-elimination-descriptor-selection
description: Use when you have a large 2D molecular descriptor matrix (e.g. 1613 Mordred
  descriptors for [M-H]-, [M+H]+ or [M+Na]+ ions) and need to shrink it to a compact
  SVR-performing subset via recursive feature elimination, stopping when RMSECV increases.
when_to_use_negative:
- The training set is not descriptor-based (e.g. you are predicting CCS from fragment
  ions, dimers like [2M+H]+, or non-Mordred representations) — this skill depends
  on a Mordred descriptor matrix
- You need a prediction tool for adducts not supported by the current CCSP 2.0 training
  sets (less common adducts, in-source fragments) — the article notes these are outside
  current scope
- You require fully automated processing without programming-language familiarity
  — the article notes use of CCSP 2.0 requires Python familiarity
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: SciKit-Learn
  role: Provides the RFECV module used to iteratively remove the five lowest-ranked
    descriptors and the GridSearchCV used to re-tune the SVR model on the RFE-selected
    subset
- name: Mordred
  role: Supplies the 1613 2D molecular descriptors that serve as the input feature
    space being reduced by RFECV
- name: CCSP 2.0
  role: Open-source Jupyter Notebook wrapper implementing the full RFE-based descriptor
    reduction and SVR CCS prediction workflow with user-curated training sets
  repo: https://github.com/facundof2016/CCSP2.0
- name: Python
  role: Runtime environment in which Mordred, SciKit-Learn and the CCSP 2.0 notebook
    execute
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:b759864c32da07f661e5d55f0a1fa61b64e1ad939b137cab40e6b2b4df7c3812
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
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# recursive-feature-elimination-descriptor-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for reducing the 1613 Mordred molecular descriptors down to an RFE-selected subset before rebuilding SVR collision cross section (CCS) prediction models, as implemented in CCSP 2.0 with the SciKit-Learn RFECV module.

## When to use

Use when you have a large 2D molecular descriptor matrix (e.g. 1613 Mordred descriptors for [M-H]-, [M+H]+ or [M+Na]+ ions) and need to shrink it to a compact SVR-performing subset via recursive feature elimination, stopping when RMSECV increases.

## When NOT to use

- The training set is not descriptor-based (e.g. you are predicting CCS from fragment ions, dimers like [2M+H]+, or non-Mordred representations) — this skill depends on a Mordred descriptor matrix
- You need a prediction tool for adducts not supported by the current CCSP 2.0 training sets (less common adducts, in-source fragments) — the article notes these are outside current scope
- You require fully automated processing without programming-language familiarity — the article notes use of CCSP 2.0 requires Python familiarity

## Inputs

- Cleaned adduct-specific training table of Z-transformed Mordred 2D molecular descriptors (from 1613 initial descriptors) paired with experimental CCS values from the McLean CCS Compendium
- Tuned linear-kernel SVR model (C and epsilon selected via GridSearchCV with 5-fold cross-validation)
- Train/test split (70% training, 30% external validation)

## Outputs

- RFE-selected descriptor subset per adduct (paper-averaged ~370, 303 and 414 descriptors for [M-H]-, [M+H]+ and [M+Na]+)
- Final re-tuned SVR model trained only on the RFE-selected descriptors
- Per-adduct calibration, cross-validation and validation error metrics (MRE, RMSE, R2)

## How to apply

After hyperparameter tuning of linear-kernel SVR models with GridSearchCV (C from 0.00390625 to 0.25, epsilon from 0.5, 1, 5; 5-fold CV), apply the SciKit-Learn RFECV module to the Z-transformed descriptor matrix. RFECV iteratively removes the five lowest-ranked molecular descriptors from the SVR model at each step, using the 5-fold cross-validated RMSECV as the stopping criterion: elimination halts once removing further descriptors causes RMSECV to increase. Record the resulting RFE-selected descriptor list per adduct (paper-averaged ~370 for [M-H]-, ~303 for [M+H]+, ~414 for [M+Na]+ of the original 1613). Rebuild the final SVR model by repeating the same GridSearchCV hyperparameter tuning on only the RFE-selected descriptors, then evaluate MRE, RMSE and R2 on the 30% external hold-out set to confirm accuracy was preserved despite descriptor reduction.

## Related tools

- **SciKit-Learn** (Provides the RFECV module used to iteratively remove the five lowest-ranked descriptors and the GridSearchCV used to re-tune the SVR model on the RFE-selected subset)
- **Mordred** (Supplies the 1613 2D molecular descriptors that serve as the input feature space being reduced by RFECV)
- **CCSP 2.0** (Open-source Jupyter Notebook wrapper implementing the full RFE-based descriptor reduction and SVR CCS prediction workflow with user-curated training sets) — https://github.com/facundof2016/CCSP2.0
- **Python** (Runtime environment in which Mordred, SciKit-Learn and the CCSP 2.0 notebook execute)

## Examples

```
# In CCSP 2.0 notebook (Jupyter/Colab): RFECV step
from sklearn.feature_selection import RFECV
rfe = RFECV(estimator=svr, step=5, scoring='neg_mean_squared_error', cv=5)
rfe.fit(X_train, y_train)
```

## Evaluation signals

- The number of selected descriptors should drop sharply from 1613 (to roughly 300–420 per adduct class in this workflow), not stay near the original count
- RMSECV should not increase at the stopping point — if it does, RFECV has not yet reached the RFE-selected subset
- Final SVR model re-tuned on RFE-selected descriptors should recover per-adduct median relative errors of ~1.25–1.87% on the 30% hold-out set (RMSEV ≈ 4.998, 5.127, 6.612 Å² for [M-H]-, [M+H]+, [M+Na]+)
- Descriptor lists should be saved per adduct so the final model can be rebuilt reproducibly from the same subset
- Verify the 70/30 train/external-validation split and Z-transform parameters (training-set mean/SD, test datapoints excluded if >1000 SD from mean) were applied before RFECV

## Limitations

- Current training sets support only [M-H]-, [M+H]+ and [M+Na]+ adducts — not dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments
- The specific training set must be defined prior to CCS prediction; RFE subsets are training-set dependent
- Current version requires SciKit-Learn V1.0.2 or later (README)
- Use requires familiarity with Python, limiting accessibility
- CCS predictions across multiple IM-MS instrument types (drift tube vs travelling wave) not yet explored

## Evidence

- [methods] RFECV iteratively removes the five lowest-ranked descriptors: [evidence span withheld — anchor status: not_found]
- [methods] RFE yields ~370/303/414 descriptors per adduct: [evidence span withheld — anchor status: not_found]
- [methods] Final SVR re-tuned on RFE-selected descriptors: [evidence span withheld — anchor status: not_found]
- [readme] README describes CCSP 2.0 as an open-source notebook for user-curated training sets: [evidence span withheld — anchor status: external_doc]
