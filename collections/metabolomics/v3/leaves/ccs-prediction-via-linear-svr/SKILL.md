---
name: ccs-prediction-via-linear-svr
description: Use when candidate structures retrieved for an IM-MS feature (e.g., PubChem
  hits within a 10 ppm mass window of an [M-H]- or [M+H]+ precursor) must be filtered
  or ranked by predicted CCS, or when CCS values must be predicted for user-curated
  training sets of molecules.
when_to_use_negative:
- Target species are dimers such as [2M+H]+ or [2M-H]-, other less common adducts,
  or in-source fragments — the current training sets do not support these
- A specific training set appropriate to the adduct/molecular class cannot be defined
  prior to prediction, or the user cannot work in a Python/Jupyter environment
- Predictions are needed across multiple IM-MS instrument platforms (e.g., drift tube
  vs travelling wave) — this has not yet been explored
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2258
tools:
- name: CCSP 2.0
  role: Open-source Jupyter Notebook that runs the linear-kernel SVR CCS prediction
    using Mordred descriptors and user-curated training sets
  repo: https://github.com/facundof2016/CCSP2.0
- name: RDKit
  role: Encodes molecular data (MOL) files from isomeric SMILES for descriptor calculation
- name: Mordred V1.2.0
  role: Calculates the 1613 2D molecular descriptors fed to the SVR model
- name: Jupyter Notebook
  role: Execution format of CCSP 2.0 (Google Colab notebook for beginners; Jupyter
    Lab notebook with Tk interface for local, large-scale use)
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:97a524a021b3de17cc431ce7a945f49e81842206843ccce2d14d8fa272d0ca8d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# ccs-prediction-via-linear-svr

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict collision cross section (CCS) values for small molecules using CCSP 2.0, an open-source Jupyter Notebook that applies linear-kernel support vector regression on 1613 Mordred 2D descriptors, enabling CCS-based filtering of false-positive metabolite annotations.

## When to use

Use when candidate structures retrieved for an IM-MS feature (e.g., PubChem hits within a 10 ppm mass window of an [M-H]- or [M+H]+ precursor) must be filtered or ranked by predicted CCS, or when CCS values must be predicted for user-curated training sets of molecules.

## When NOT to use

- Target species are dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments — the current training sets do not support these
- A specific training set appropriate to the adduct/molecular class cannot be defined prior to prediction, or the user cannot work in a Python/Jupyter environment
- Predictions are needed across multiple IM-MS instrument platforms (e.g., drift tube vs travelling wave) — this has not yet been explored

## Inputs

- Isomeric SMILES strings or InChI identifiers for candidate molecules (e.g., PubChem CIDs resolved via PubChemPy)
- MOL files encoded from SMILES with RDKit
- User-curated training set of experimental CCS values (e.g., McLean Unified CCS Compendium subset for the adduct of interest)
- Experimentally measured CCS value (Å2) and mass error window (ppm) for the target IM-MS feature

## Outputs

- Predicted CCS values (Å2) for each candidate molecule
- Filtered annotation candidate list retaining molecules within the ΔCCS threshold (e.g., 2.8%, or ±2% of experimental CCS)

## How to apply

Convert each molecule's neutral InChI (via the PubChem Identifier Exchange Service) or isomeric SMILES (retrieved via PubChemPy) into MOL files with RDKit, then compute 1613 2D molecular descriptors with Mordred V1.2.0. Select a training set matched to the adduct of interest — e.g., the 170 [M-H]-, 155 [M+H]+ or 138 [M+Na]+ entries from the McLean Unified CCS Compendium — before running prediction, since CCSP 2.0 supports user-customized training sets. Run the linear-kernel SVR model (Scikit-learn V1.0.2 or later, per the README) via the Google Colab notebook (recommended for <10,000 molecules) or the locally hosted Jupyter Lab notebook for larger or workflow-integrated predictions. Judge predictions against the reported median relative errors of 1.25% ([M-H]-), 1.73% ([M+H]+) and 1.87% ([M+Na]+); filter annotation candidates with a ΔCCS threshold of 2.8% (or ~±2% of the experimental CCS, as in the isoorotic acid case: predicted 119.3 vs experimental 120.22 Å2) combined with a 10 ppm mass-error window.

## Related tools

- **CCSP 2.0** (Open-source Jupyter Notebook that runs the linear-kernel SVR CCS prediction using Mordred descriptors and user-curated training sets) — https://github.com/facundof2016/CCSP2.0
- **RDKit** (Encodes molecular data (MOL) files from isomeric SMILES for descriptor calculation)
- **Mordred V1.2.0** (Calculates the 1613 2D molecular descriptors fed to the SVR model)
- **Jupyter Notebook** (Execution format of CCSP 2.0 (Google Colab notebook for beginners; Jupyter Lab notebook with Tk interface for local, large-scale use)) — https://github.com/facundof2016/CCSP2.0

## Examples

```
jupyter lab "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"  # or upload 'CCSP 2.0 - CCS Prediction for Google Colab.ipynb' at https://colab.research.google.com/
```

## Evaluation signals

- Median relative prediction error should fall near 1.25% for [M-H]-, 1.73% for [M+H]+ and 1.87% for [M+Na]+ when validated against McLean CCS Compendium test measurements
- At a ΔCCS threshold of 2.8% with a 10 ppm mass-error window, filtering should remove ~36.1% of incorrect structures while retaining 100% of correct annotations
- Reproducing the isoorotic acid case should return 221 PubChem candidates at m/z 155.0093 within 10 ppm, of which exactly 32 are retained at a ±2% CCS window (86% false positives removed; correct annotation CCS predicted 119.3 vs experimental 120.22 Å2)
- Environment check: the notebook runs only with Scikit-learn V1.0.2 or later (see requirements.txt)
- Descriptor count invariant: exactly 1613 2D Mordred descriptors computed per MOL file before prediction

## Limitations

- Training sets do not cover dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments
- The specific training set must be defined prior to CCS prediction
- Use requires familiarity with a programming language such as Python, limiting accessibility (Google Colab mitigates installation but disconnects after ~20 min idle and limits sessions to ~12 hours)
- CCS predictions across multiple IM-MS instrument types (drift tube, travelling wave) and multi-adduct/fragment-ion prediction are unexplored future work
- Current version requires Scikit-learn V1.0.2 or later

## Evidence

- [readme] readme-scope: [evidence span withheld — anchor status: external_doc]
- [readme] readme-version: [evidence span withheld — anchor status: external_doc]
