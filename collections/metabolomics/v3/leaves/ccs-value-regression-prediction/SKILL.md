---
name: ccs-value-regression-prediction
description: Use when a metabolite annotation list (e.g., PubChem candidates within
  10 ppm of an exact mass) needs CCS values predicted for [M-H]-, [M+H]+ or [M+Na]+
  adducts to enable ΔCCS-threshold filtering.
when_to_use_negative:
- The analytes are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments
  — the current training sets do not support them
- CCS predictions are needed across multiple IM-MS instrument types (drift tube vs.
  travelling wave) — this cross-platform generalization is explicitly left as future
  work
- The user cannot define a specific training set prior to prediction, since CCSP requires
  the training set to be defined beforehand
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook implementing the linear support vector regression
    CCS prediction with user-curated training sets
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred V1.2.0
  role: Python package used to calculate the 1613 2D molecular descriptors that serve
    as SVR input features
- name: RDKit
  role: Encodes molecular data (MOL) files for each SMILES prior to descriptor calculation
- name: Mordred
  role: Descriptor library (1613 molecular descriptors) included with CCSP 2.0 for
    featurizing molecules
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:3c579ad0213554aa89fc0da125b61213dfc18eacd006ba3fb5ec7640f59b3919
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# ccs-value-regression-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict gas-phase collision cross section (CCS) values with a linear-kernel support vector regression model (CCSP 2.0) trained on user-curated IM-MS data, using 1613 Mordred 2D molecular descriptors, so predicted CCS can be used to filter false-positive metabolite annotations.

## When to use

Use when a metabolite annotation list (e.g., PubChem candidates within 10 ppm of an exact mass) needs CCS values predicted for [M-H]-, [M+H]+ or [M+Na]+ adducts to enable ΔCCS-threshold filtering.

## When NOT to use

- The analytes are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments — the current training sets do not support them
- CCS predictions are needed across multiple IM-MS instrument types (drift tube vs. travelling wave) — this cross-platform generalization is explicitly left as future work
- The user cannot define a specific training set prior to prediction, since CCSP requires the training set to be defined beforehand

## Inputs

- Neutral InChI strings or isomeric SMILES for each candidate molecule
- MOL files encoded from SMILES via RDKit
- Training set of drift tube CCS measurements (e.g., McLean Unified CCS Compendium entries) for the target adduct such as [M-H]-
- Candidate annotation lists from PubChem queries at a 10 ppm exact-mass tolerance

## Outputs

- Predicted CCS values for every candidate molecule
- Predictions exported from the CCSP 2.0 notebook for downstream ΔCCS filtering (e.g., filtering_stats.csv)

## How to apply

Represent each molecule as SMILES converted from neutral InChI strings, encode MOL files with RDKit, and compute 1613 2D molecular descriptors with Mordred V1.2.0. Train a linear-kernel SVR (CCSP 2.0, requiring sklearn V1.0.2+) on a user-curated training set drawn from the McLean Unified CCS Compendium, e.g., a 70:30 split of deprotonated [M-H]- compounds such as the 397-calibration-compound set. Feed the descriptor matrix for each candidate molecule to the model to obtain predicted CCS values, then compare predicted vs. measured or expected CCS at a ΔCCS threshold (e.g., 2.8-3%) against a mass-error window of 10 ppm. Choose the training set prior to prediction — CCSP allows customization to specific adducts or molecular classes — and run either the Google Colab notebook (small-scale, <10,000 molecules) or the local Jupyter Lab notebook with Tk interface for large training/target sets.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook implementing the linear support vector regression CCS prediction with user-curated training sets) — https://github.com/facundof2016/CCSP2.0
- **Mordred V1.2.0** (Python package used to calculate the 1613 2D molecular descriptors that serve as SVR input features)
- **RDKit** (Encodes molecular data (MOL) files for each SMILES prior to descriptor calculation)
- **Mordred** (Descriptor library (1613 molecular descriptors) included with CCSP 2.0 for featurizing molecules)

## Examples

```
Open 'CCSP 2.0 - CCS Prediction in Jupyter.ipynb' in Jupyter Lab (Python 3, sklearn >= V1.0.2) and follow the embedded instructions to train the linear SVR on the curated training set and predict CCS for the target descriptor matrix.
```

## Evaluation signals

- Median relative prediction errors should fall near the reported benchmarks: 1.25% for [M-H]-, 1.73% for [M+H]+, 1.87% for [M+Na]+ against the McLean CCS Compendium
- When used for filtering at ΔCCS 2.8% with a 10 ppm mass error, CCSP predictions should remove ~36.1% of incorrect structures while retaining 100% of correct annotations
- The notebook must run with sklearn V1.0.2 or later; check requirements.txt before execution
- On the 162 deprotonated test-compound benchmark, ΔCCS 3% filtering of 10 ppm PubChem candidate lists (598,208 candidates total) must retain 76.5% of true annotations and filter a median 47.3% of candidates (24.9% for same-formula candidates)

## Limitations

- Current training sets do not support dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments found in ESI-MS experiments
- The specific training set must be defined prior to CCS prediction
- Use requires familiarity with a programming language such as Python, limiting accessibility
- Effect of experimental sources of variance on the CCS filtering threshold choice has not yet been investigated; multi-adduct filtering and fragment-ion CCS prediction are future work
- Google Colab limits continuous notebook operation to 12 hours and disconnects after ~20 minutes idle; large training/target sets require the locally hosted Jupyter Lab version

## Evidence

- [readme] The current version of CCSP 2.0 requires Sklearn V1.0.2 or later.: [evidence span withheld — anchor status: external_doc]
- [readme] If you plan to predict CCS values using large training or target sets or if you plan to integrate the code into a larger analysis workflow, the locally hosted Jupyter Lab version is recommended.: [evidence span withheld — anchor status: external_doc]
