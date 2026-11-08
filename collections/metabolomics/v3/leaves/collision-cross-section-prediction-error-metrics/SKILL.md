---
name: collision-cross-section-prediction-error-metrics
description: Use when validating or reproducing CCS prediction accuracy — computing
  median relative error, RMSEV, and cumulative fractions of predictions within 3%
  and 5% error for linear-SVR CCS models on [M-H]-, [M+H]+ and [M+Na]+ test sets.
when_to_use_negative:
- You need to predict CCS values or filter unknown metabolite annotations at a ΔCCS
  threshold of 2.8% with 10 ppm mass error — use the CCSP 2.0 prediction workflow,
  not the validation-metric reproduction
- Molecules of interest are dimers such as [2M+H]+ or [2M-H]-, less common adducts,
  or in-source fragments found in ESI-MS experiments — CCSP 2.0 training sets do not
  support these
- No experimental drift-tube CCS ground truth exists for the compounds (only predictions)
  — MRE, RMSEV and within-3%/5% fractions cannot be computed without measured reference
  values
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook whose linear-SVR CCS models (101 randomly-selected
    test sets) produce the validation metrics reproduced by this skill
  repo: https://github.com/facundof2016/CCSP2.0
- name: RDKit
  role: Encodes molecular data (MOL) files for each SMILES before descriptor calculation
- name: Mordred V1.2.0
  role: Calculates the 1613 2D molecular descriptors used as SVR input features
- name: SciKit-Learn
  role: Trains the linear-kernel SVR with L2 regularization, runs 5-fold GridSearchCV
    over C and epsilon, and performs RFECV descriptor elimination
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:c58de78e2f11dee6022051d58f4003f680a3e0ea5f81fa13222146d5fa07010c
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# collision-cross-section-prediction-error-metrics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces CCSP 2.0 validation-performance metrics — median relative error (MRE), RMSE of validation (RMSEV), and cumulative fractions of predictions within 3% and 5% error — for linear-SVR CCS models on [M-H]-, [M+H]+ and [M+Na]+ test sets drawn from the McLean Unified CCS Compendium via 101 random 70/30 splits.

## When to use

Use when validating or reproducing CCS prediction accuracy — computing median relative error, RMSEV, and cumulative fractions of predictions within 3% and 5% error for linear-SVR CCS models on [M-H]-, [M+H]+ and [M+Na]+ test sets.

## When NOT to use

- You need to predict CCS values or filter unknown metabolite annotations at a ΔCCS threshold of 2.8% with 10 ppm mass error — use the CCSP 2.0 prediction workflow, not the validation-metric reproduction
- Molecules of interest are dimers such as [2M+H]+ or [2M-H]-, less common adducts, or in-source fragments found in ESI-MS experiments — CCSP 2.0 training sets do not support these
- No experimental drift-tube CCS ground truth exists for the compounds (only predictions) — MRE, RMSEV and within-3%/5% fractions cannot be computed without measured reference values

## Inputs

- Cleaned McLean Unified CCS Compendium adduct datasets: 567 [M-H]-, 518 [M+H]+ and 461 [M+Na]+ drift-tube CCS measurements with m/z and structures (neutral InChI converted to isomeric SMILES)
- RDKit-encoded MOL files for each SMILES
- Mordred V1.2.0 2D descriptor table (1613 descriptors) after removal of constant and inapplicable descriptors
- CCSP 2.0 Jupyter notebook environment with sklearn V1.0.2+ (requirements.txt)

## Outputs

- Per-split RMSEV and MRE for each of 101 random 70/30 splits, per adduct
- Median-RMSEV model metrics per adduct: median relative error (%), RMSEV (Å²), and cumulative fractions of validation predictions within 3% and 5% error
- ccsp2_validation_metrics.csv containing all per-split and median-model metrics for the three adducts

## How to apply

Scrub the McLean Unified CCS Compendium adduct datasets (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions) per the paper's rules: average duplicates, exclude entries whose m/z deviates >1 mDa, elements with <5 entries, and mass ranges incompatible with DeepCCS/AllCCS/CCSBase. Convert neutral InChI strings to isomeric SMILES (PubChem Identifier Exchange), encode MOL files with RDKit, compute 1613 2D descriptors with Mordred V1.2.0, discard descriptors constant across calibration molecules or inapplicable, and Z-transform with training-set statistics (excluding validation points >1000 SD from the mean). Run 101 iterations: each randomly assigns 70% calibration / 30% validation, trains a linear-kernel SVR (L2) tuning C over seven grid values (0.00390625–0.25) and epsilon over (0.5, 1, 5) via 5-fold GridSearchCV, and applies RFECV removing the five lowest-ranked descriptors per iteration until RMSECV increases. Compute RMSEV and MRE per split, select the median-RMSEV split per adduct, and report median relative error, RMSEV, and cumulative fractions of validation predictions within 3% and 5% error, writing per-split and median-model metrics to ccsp2_validation_metrics.csv. Accept the reproduction when the median-model metrics match the reported values: MRE of 1.251%/1.729%/1.874% and RMSEV of 4.998/5.127/6.612 Å² for [M-H]-/[M+H]+/[M+Na]+, with 78%/70%/74% of predictions within 3% and 92%/92%/90% within 5% error; run under sklearn V1.0.2 or later per requirements.txt.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook whose linear-SVR CCS models (101 randomly-selected test sets) produce the validation metrics reproduced by this skill) — https://github.com/facundof2016/CCSP2.0
- **RDKit** (Encodes molecular data (MOL) files for each SMILES before descriptor calculation)
- **Mordred V1.2.0** (Calculates the 1613 2D molecular descriptors used as SVR input features)
- **SciKit-Learn** (Trains the linear-kernel SVR with L2 regularization, runs 5-fold GridSearchCV over C and epsilon, and performs RFECV descriptor elimination)

## Examples

```
jupyter lab 'CCSP 2.0 - CCS Prediction in Jupyter.ipynb'
```

## Evaluation signals

- Median-model MRE per adduct reproduces 1.251%/1.729%/1.874% for [M-H]-/[M+H]+/[M+Na]+
- Median-RMSEV values land at 4.998/5.127/6.612 Å² respectively for [M-H]-/[M+H]+/[M+Na]+
- Cumulative prediction-error fractions match 78%/70%/74% within 3% error and 92%/92%/90% within 5% error
- Test-set sizes per adduct equal 170/155/138 ions, summing to the 463 test CCS measurements described in the paper
- ccsp2_validation_metrics.csv exists and contains per-split plus median-model rows for all three adducts

## Limitations

- CCSP 2.0 training sets do not support dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments found in ESI-MS experiments
- The Google Colab route caps continuous operation at 12 hours and disconnects after ~20 minutes idle; the locally hosted Jupyter Lab version is recommended for large training/target sets or integrating into larger workflows
- Requires Sklearn V1.0.2 or later and familiarity with Python, which limits accessibility
- Median-RMSEV selection across 101 random splits means single-seed reruns will vary; compare median statistics rather than individual split values
- CCS prediction across multiple IM-MS instrument types (drift tube vs travelling wave) is explicitly left as future work

## Evidence

- [readme] The current version of CCSP 2.0 requires Sklearn V1.0.2 or later. A full list of package requirements is available in the requirements.txt file.: [evidence span withheld — anchor status: external_doc]
