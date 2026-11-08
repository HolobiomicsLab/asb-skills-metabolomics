---
name: web-server-and-cli-prediction-harnessing
description: Use when benchmarking multiple CCS prediction tools (e.g. CCSP 2.0, CCSbase,
  AllCCS, DeepCCS) on the same molecule set and adduct type to rank them by median
  MRE/RMSE or to decide which predictor to trust for annotation filtering.
when_to_use_negative:
- When the training set does not cover the target adduct — the paper's training sets
  do not support dimers such as [2M+H]+ or [2M-H]-, less common adducts, or in-source
  fragments found in ESI-MS experiments
- When no Python environment or browser-based notebook access is available and only
  a single predictor's output is needed — the web-server round-trips (CCSbase, AllCCS)
  and DeepCCS CLI require manual submission or installation
- 'When the goal is CCS prediction across different IM-MS instrument types (drift
  tube vs travelling wave): the paper notes this has not yet been explored, so cross-instrument
  comparisons are out of scope'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: CCSP 2.0
  role: Locally-trained linear-kernel SVR predictor (Mordred descriptors, grid-searched
    C/epsilon with 5-fold CV, RFE descriptor selection) whose withheld-split predictions
    are compared against the external tools
  repo: https://github.com/facundof2016/CCSP2.0
- name: AllCCS
  role: External web-server CCS predictor that receives the same isomeric SMILES test
    split as the other tools for head-to-head benchmarking
- name: CCSbase
  role: External web-server CCS predictor submitting the identical SMILES test set
    to produce comparable predictions
- name: DeepCCS
  role: Machine-learning CCS predictor invoked through its built-in command-line model
    on the same test-set SMILES
- name: Mordred
  role: Python package (V1.2.0) that computes the 1613 2D molecular descriptors from
    RDKit MOL files to feed the CCSP 2.0 SVR branch of the harness
- name: RDKit
  role: Encodes MOL files from each isomeric SMILES before descriptor calculation
- name: PubChem Identifier Exchange Service
  role: Converts neutral InChI strings from the McLean Compendium into the isomeric
    SMILES that all four predictors consume
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:f098b51b1bf47b9d4452f098179da6acf4e9be8a4d9f86b5c533215c97f4082e
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
  - AllCCS
  - CCSP 2.0
  - CCSbase
  - DeepCCS
  - Mordred
  - PubChem Identifier Exchange Service
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# web-server-and-cli-prediction-harnessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harnesses identical SMILES inputs into heterogeneous CCS prediction back-ends — AllCCS and CCSbase web servers, the DeepCCS command-line model, and the locally-trained CCSP 2.0 SVR — so that predicted drift-tube CCS values can be compared side-by-side per adduct.

## When to use

Use when benchmarking multiple CCS prediction tools (e.g. CCSP 2.0, CCSbase, AllCCS, DeepCCS) on the same molecule set and adduct type to rank them by median MRE/RMSE or to decide which predictor to trust for annotation filtering.

## When NOT to use

- When the training set does not cover the target adduct — the paper's training sets do not support dimers such as [2M+H]+ or [2M-H]-, less common adducts, or in-source fragments found in ESI-MS experiments
- When no Python environment or browser-based notebook access is available and only a single predictor's output is needed — the web-server round-trips (CCSbase, AllCCS) and DeepCCS CLI require manual submission or installation
- When the goal is CCS prediction across different IM-MS instrument types (drift tube vs travelling wave): the paper notes this has not yet been explored, so cross-instrument comparisons are out of scope

## Inputs

- Neutral InChI strings from the McLean Unified CCS Compendium (converted to isomeric SMILES)
- Experimentally measured drift-tube CCS values per adduct ([M-H]-, [M+H]+, [M+Na]+)
- 70:30 calibration/test split assignments shared across all compared tools (101 random splits)
- Mordred V1.2.0 descriptor matrix (1613 2D descriptors) computed from RDKit MOL files for CCSP 2.0 training

## Outputs

- CCS predictions per tool (CCSP 2.0, CCSbase, AllCCS, DeepCCS) for each test-set SMILES
- Per-split MRE and RMSE values per tool and adduct
- Median MRE and median RMSE rankings across splits (e.g. [M-H]- MRE: 1.251%, 1.427%, 1.855%, 1.871%)

## How to apply

Convert every molecule's neutral InChI string to an isomeric SMILES via PubChem's Identifier Exchange Service, then encode MOL files with RDKit and compute the 1613 2D Mordred V1.2.0 descriptors for the CCSP 2.0 branch of the harness. Draw a consistent 30% test split (e.g. from the 567 [M-H]-, 518 [M+H]+, or 461 [M+Na]+ ions of the McLean Unified CCS Compendium) and submit the identical SMILES to the AllCCS and CCSbase web servers while running DeepCCS through its built-in command-line model on the same inputs, so no tool sees a different representation. Train the CCSP 2.0 linear-kernel SVR on the remaining 70% (grid-searching C and epsilon with 5-fold cross-validation plus RFE descriptor selection) and predict the withheld 30%. Compute MRE and RMSE against the experimental McLean Compendium CCS values per tool, per adduct, and aggregate across splits (the paper uses 101 splits) to obtain median metrics for ranking — e.g. [M-H]- median MRE of CCSP 2.0 1.251%, CCSbase 1.427%, AllCCS 1.855%, DeepCCS 1.871%.

## Related tools

- **CCSP 2.0** (Locally-trained linear-kernel SVR predictor (Mordred descriptors, grid-searched C/epsilon with 5-fold CV, RFE descriptor selection) whose withheld-split predictions are compared against the external tools) — https://github.com/facundof2016/CCSP2.0
- **AllCCS** (External web-server CCS predictor that receives the same isomeric SMILES test split as the other tools for head-to-head benchmarking)
- **CCSbase** (External web-server CCS predictor submitting the identical SMILES test set to produce comparable predictions)
- **DeepCCS** (Machine-learning CCS predictor invoked through its built-in command-line model on the same test-set SMILES)
- **Mordred** (Python package (V1.2.0) that computes the 1613 2D molecular descriptors from RDKit MOL files to feed the CCSP 2.0 SVR branch of the harness)
- **RDKit** (Encodes MOL files from each isomeric SMILES before descriptor calculation)
- **PubChem Identifier Exchange Service** (Converts neutral InChI strings from the McLean Compendium into the isomeric SMILES that all four predictors consume)

## Examples

```
python -c "from rdkit import Chem; from mordred import Calculator; m=Chem.MolFromSmiles('OC(=O)Cc1ccc(O)cc1'); print(Calculator()(m))"
```

## Evaluation signals

- Median MRE across 101 splits must reproduce published values (e.g. [M-H]-: CCSP 2.0 1.251%, CCSbase 1.427%, AllCCS 1.855%, DeepCCS 1.871%) within sampling variability
- Median RMSE must match published ranges, e.g. [M-H]- RMSE of 4.876, 7.359, 9.919, 6.068 Å² for CCSP 2.0, CCSbase, AllCCS, DeepCCS
- All four predictors must receive byte-identical isomeric SMILES for the same split so ranking differences reflect model quality, not input differences
- CCSP 2.0 branch must be trained only on the 70% calibration portion of each split, with the 30% test portion never used in grid-searched C/epsilon tuning (5-fold CV) or RFE selection
- Predicted CCS values must be compared against the experimentally-measured McLean Compendium CCS values for the same adduct type

## Limitations

- Web-server access (CCSbase, AllCCS) and DeepCCS CLI require manual submission or installation; Google Colab route disconnects after ~20 minutes idle or 12 hours continuous
- Training sets do not cover dimers ([2M+H]+, [2M-H]-), other less common adducts, or in-source fragments from ESI-MS experiments
- The specific training set must be defined before prediction; Sklearn V1.0.2 or later is required
- CCS prediction across multiple IM-MS instrument types (drift tube vs travelling wave) is unexplored; effect of experimental variance on CCS filtering threshold choice is not yet characterized
- Using CCSP 2.0 requires familiarity with Python, which limits accessibility

## Evidence

- [other] Submit identical SMILES to AllCCS and CCSbase web servers and DeepCCS CLI: [evidence span withheld — anchor status: not_found]
- [readme] CCSP 2.0 requires Sklearn V1.0.2 or later: [evidence span withheld — anchor status: external_doc]
