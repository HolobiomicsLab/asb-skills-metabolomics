---
name: cross-validation-split-sampling-and-median-model-selection
description: Use when training CCS-prediction SVR models on McLean Unified CCS Compendium
  adduct data ([M-H]-, [M+H]+, [M+Na]+) and you need unbiased split-level performance
  reporting via a median-RMSEV model rather than a single arbitrary split.
when_to_use_negative:
- The training set does not support dimers ([2M+H]+, [2M-H]-), less common adducts,
  or in-source fragments found in ESI-MS experiments, so this split protocol is inappropriate
  for those ion types
- A single fixed train/test split is already mandated by the study design — re-sampling
  101 times would not answer that question
- Predictions across multiple IM-MS instrument types (drift tube vs travelling wave)
  are needed — this protocol was validated only on the compiled drift-tube CCS data
  and cross-instrument behavior is unexplored
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CCSP 2.0
  role: Jupyter Notebook implementation containing the 101-iteration split sampling,
    GridSearchCV tuning, RFECV descriptor elimination, and median-RMSEV model selection
    logic
  repo: https://github.com/facundof2016/CCSP2.0
- name: SciKit-Learn
  role: Provides the linear-kernel SVR (L2 regularization), 5-fold GridSearchCV over
    C (0.00390625-0.25) and epsilon (0.5, 1, 5), and RFECV descriptor ranking/elimination
- name: Mordred V1.2.0
  role: Calculates the 1613 2D molecular descriptors that feed the per-split calibration
    matrices before RFECV pruning
- name: RDKit
  role: Encodes molecular data (MOL) files for each SMILES so descriptors can be computed
    per molecule
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:1362680107e79081dc3c237e6a8af324fe55e5a5ff63f56bb7ae9c258ec725da
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# cross-validation-split-sampling-and-median-model-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A repeat-random-split cross-validation and median-model selection protocol from CCSP 2.0: 101 randomized 70/30 calibration/validation splits of McLean CCS Compendium adduct datasets, SVR hyperparameter tuning via 5-fold GridSearchCV, and reporting metrics from the median-RMSEV model.

## When to use

Use when training CCS-prediction SVR models on McLean Unified CCS Compendium adduct data ([M-H]-, [M+H]+, [M+Na]+) and you need unbiased split-level performance reporting via a median-RMSEV model rather than a single arbitrary split.

## When NOT to use

- The training set does not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments, so this split protocol is inappropriate for those ion types
- A single fixed train/test split is already mandated by the study design — re-sampling 101 times would not answer that question
- Predictions across multiple IM-MS instrument types (drift tube vs travelling wave) are needed — this protocol was validated only on the compiled drift-tube CCS data and cross-instrument behavior is unexplored

## Inputs

- Cleaned adduct datasets from the McLean Unified CCS Compendium (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions) with CCS values
- 1613 2D Mordred V1.2.0 descriptors per molecule (constant/inapplicable descriptors removed, Z-transformed with training-set statistics)
- RDKit-encoded MOL files derived from isomeric SMILES (converted from neutral InChI strings via PubChem Identifier Exchange Service)

## Outputs

- Per-split RMSEV and MRE values for all 101 iterations per adduct
- Median-RMSEV model metrics: median relative error (%), RMSEV (Å²), and cumulative fractions of validation predictions within 3% and 5% error
- ccsp2_validation_metrics.csv containing per-split and median-model metrics for the three adducts

## How to apply

For each adduct dataset ([M-H]- with 567 ions, [M+H]+ with 518, [M+Na]+ with 461 after scrubbing), run 101 iterations, each randomly assigning 70% of molecules to calibration and 30% to validation. In each iteration, tune a linear-kernel SVR (L2 regularization) via 5-fold cross-validation with GridSearchCV over seven C values from 0.00390625 to 0.25 and epsilon in (0.5, 1, 5), then apply RFECV removing the five lowest-ranked descriptors per iteration until RMSECV increases. Compute RMSEV and MRE on each iteration's validation set, then select the split yielding the median RMSEV for closer inspection, reporting median relative error, RMSEV, and cumulative fractions of predictions within 3% and 5% error. The rationale: aggregating 101 random splits and reporting the median model avoids cherry-picking a single favorable split while remaining representative of typical performance.

## Related tools

- **CCSP 2.0** (Jupyter Notebook implementation containing the 101-iteration split sampling, GridSearchCV tuning, RFECV descriptor elimination, and median-RMSEV model selection logic) — https://github.com/facundof2016/CCSP2.0
- **SciKit-Learn** (Provides the linear-kernel SVR (L2 regularization), 5-fold GridSearchCV over C (0.00390625-0.25) and epsilon (0.5, 1, 5), and RFECV descriptor ranking/elimination)
- **Mordred V1.2.0** (Calculates the 1613 2D molecular descriptors that feed the per-split calibration matrices before RFECV pruning)
- **RDKit** (Encodes molecular data (MOL) files for each SMILES so descriptors can be computed per molecule)

## Examples

```
GridSearchCV(SVR(kernel='linear'), param_grid={'C': [0.00390625, 0.0078125, 0.015625, 0.03125, 0.0625, 0.125, 0.25], 'epsilon': [0.5, 1, 5]}, cv=5)
```

## Evaluation signals

- Reproduced median-model RMSEV should match 4.998 / 5.127 / 6.612 Å² for [M-H]- / [M+H]+ / [M+Na]+ test compounds
- Reproduced median MRE should match 1.251% / 1.729% / 1.874% across the three adduct sets
- Fractions of validation predictions within 3% error should fall at 78% / 70% / 74%, and within 5% error at 92% / 92% / 90%
- Exactly 101 splits per adduct must be recorded, each with 70/30 calibration/validation assignment and per-split RMSEV/MRE values
- RFECV within each split must terminate when removing five more lowest-ranked descriptors causes RMSECV to increase

## Limitations

- The current version of CCSP 2.0 requires Sklearn V1.0.2 or later; older scikit-learn versions may change GridSearchCV/RFECV behavior
- Test points more than 1000 standard deviations from the training mean are excluded from Z-transform scaling, which can shrink the validation set
- Google Colab operation is capped at 12 continuous hours and disconnects after ~20 minutes idle, so large training/target sets require the locally hosted Jupyter Lab notebook
- The specific training set must be defined prior to CCS prediction; the protocol cannot be run adduct-agnostically

## Evidence

- [methods] 101-iteration protocol with median selection: [evidence span withheld — anchor status: not_found]
- [readme] Scikit-learn version requirement: [evidence span withheld — anchor status: external_doc]
