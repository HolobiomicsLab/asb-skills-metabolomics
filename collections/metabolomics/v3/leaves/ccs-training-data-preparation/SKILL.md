---
name: ccs-training-data-preparation
description: Use when building or customizing a CCS prediction training set from McLean
  CCS Compendium-style records (neutral InChI strings, per-adduct CCS values) before
  fitting linear SVR models.
when_to_use_negative:
- The query molecules are dimers such as [2M+H]+ or [2M-H]-, other less common adducts,
  or in-source fragments — the current CCSP 2.0 training sets do not support them
- Input is already a cleaned, scaled descriptor table ready for direct SVR/RFECV training
  — re-preparation would duplicate the InChI-to-descriptor pipeline
- The intended use is CCS prediction on a pre-trained model rather than creating or
  customizing a training set
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: CCSP 2.0
  role: Jupyter Notebook tool whose training-set customization workflow this preparation
    step feeds
  repo: https://github.com/facundof2016/CCSP2.0
- name: PubChem Identifier Exchange Service
  role: Converts neutral InChI strings to isomeric SMILES specifications
- name: RDKit
  role: Encodes molecular data (MOL) files for each SMILES
- name: Mordred V1.2.0
  role: Calculates the 1613 2D molecular descriptors from each MOL file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:58d193f2e90b1c1a0aa705a6cfd24577f3b4dcd8b47235a09138d2ee7e01e2eb
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
  - Numpy
  - Pandas
  - PubChem Identifier Exchange Service
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# ccs-training-data-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare molecular training data for CCSP 2.0 collision cross section prediction: convert InChI strings to isomeric SMILES, encode MOL files with RDKit, compute 1613 2D descriptors with Mordred, and split/scale the cleaned per-adduct dataset for SVR training.

## When to use

Use when building or customizing a CCS prediction training set from McLean CCS Compendium-style records (neutral InChI strings, per-adduct CCS values) before fitting linear SVR models.

## When NOT to use

- The query molecules are dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments — the current CCSP 2.0 training sets do not support them
- Input is already a cleaned, scaled descriptor table ready for direct SVR/RFECV training — re-preparation would duplicate the InChI-to-descriptor pipeline
- The intended use is CCS prediction on a pre-trained model rather than creating or customizing a training set

## Inputs

- Neutral InChI strings for calibration molecules from the McLean Unified CCS Compendium
- Per-adduct experimental CCS values ([M-H]-, [M+H]+, [M+Na]+)
- Mordred-calculated 1613 2D molecular descriptor table

## Outputs

- Cleaned per-adduct descriptor matrices with constant and inapplicable descriptors removed
- 70/30 train/external-validation split (random assignment)
- Z-transformed descriptor values scaled with training-set mean and standard deviation
- Selected-descriptor lists carried forward to SVR model building

## How to apply

Convert each molecule's neutral InChI string to its isomeric SMILES via the PubChem Identifier Exchange Service, then use RDKit to encode MOL files for each SMILES. Compute 1613 2D molecular descriptors with the Mordred Python package (V1.2.0), and discard descriptors that are constant across calibration molecules or inapplicable (e.g. division-by-zero bond-path descriptors). Randomly assign 70% of molecules to training and 30% to external validation, working per adduct (e.g. 463 test CCS measurements spanning 170 [M-H]-, 155 [M+H]+ and 138 [M+Na]+ adducts). Z-transform the remaining descriptors using only the training-set mean and standard deviation, and exclude any test datapoint exceeding 1000 standard deviations from its mean to guard against extreme scaling artifacts. The resulting clean descriptor tables and selected-descriptor lists feed the linear-kernel SVR training and RFECV feature-elimination steps.

## Related tools

- **CCSP 2.0** (Jupyter Notebook tool whose training-set customization workflow this preparation step feeds) — https://github.com/facundof2016/CCSP2.0
- **PubChem Identifier Exchange Service** (Converts neutral InChI strings to isomeric SMILES specifications)
- **RDKit** (Encodes molecular data (MOL) files for each SMILES)
- **Mordred V1.2.0** (Calculates the 1613 2D molecular descriptors from each MOL file)

## Examples

```
jupyter lab "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"
```

## Evaluation signals

- Descriptor table contains up to 1613 2D Mordred descriptor columns before cleaning; after cleaning, constant-across-molecules and inapplicable (division-by-zero bond-path) descriptors are absent
- Train/external-validation split is 70% / 30% by random assignment, stratified per adduct (170 [M-H]-, 155 [M+H]+, 138 [M+Na]+ test adducts = 463 CCS measurements)
- Z-transformation uses training-set mean and standard deviation only; no test datapoint exceeding 1000 standard deviations from its mean remains in the set
- Downstream SVR/RFECV achieves median relative errors of 1.25, 1.73 and 1.87% for [M-H]-, [M+H]+ and [M+Na]+ adducts (below the ~2% target) on the hold-out set
- Environment check: scikit-learn V1.0.2 or later is installed per the repository requirements.txt

## Limitations

- Current training sets do not support dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments found in ESI-MS experiments
- The specific training set to be used must be defined prior to CCS prediction
- Using CCSP 2.0 requires familiarity with a programming language such as Python, limiting accessibility
- Google Colab execution is capped at ~12 hours continuous operation and disconnects after ~20 minutes idle; large training sets should use the locally hosted Jupyter Lab version

## Evidence

- [readme] Collision Cross Section Predictor 2.0 is an open source Python notebook intended to help ion-mobility scientists predict collision cross sections with user-curated training sets.: [evidence span withheld — anchor status: external_doc]
- [readme] NOTE | The current version of CCSP 2.0 requires Sklearn V1.0.2 or later.: [evidence span withheld — anchor status: external_doc]
