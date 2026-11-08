---
name: mclean-compendium-sampled-split-generation
description: Use when benchmarking multiple CCS prediction tools (e.g., CCSP 2.0,
  CCSbase, AllCCS, DeepCCS) on the McLean Compendium and you need reproducible, tool-shared
  random test splits with aggregate median MRE/RMSE metrics.
when_to_use_negative:
- When you only need a single train/test partition for model development rather than
  a multi-tool benchmark with median aggregation over 101 splits
- When the molecules of interest are dimers ([2M+H]+, [2M-H]-), less common adducts,
  or in-source fragments, since the compendium training sets used here do not support
  them
tools:
- name: CCSP 2.0
  role: Local SVR-based CCS predictor trained on the 70% calibration portion of each
    split and evaluated on the shared 30% test set
  repo: https://github.com/facundof2016/CCSP2.0
- name: CCSbase
  role: External web-server predictor receiving the identical isomeric SMILES test
    sets from each split
- name: AllCCS
  role: External web-server predictor receiving the identical isomeric SMILES test
    sets from each split
- name: DeepCCS
  role: Built-in command-line CCS model applied to the identical isomeric SMILES test
    sets from each split
- name: Python
  role: Language used to implement the split generation, descriptor calculation, and
    metric aggregation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:2872d96ae05f9b6d6dc55778750e82c28f5539b6f9b7f375684bfcda59b76d13
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
  - Python
  - RDKit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# mclean-compendium-sampled-split-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates 101 random 70:30 calibration/test splits of the cleaned McLean Unified CCS Compendium adduct data sets (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions) so that identical test sets can be shared across CCS prediction tools for fair benchmarking.

## When to use

Use when benchmarking multiple CCS prediction tools (e.g., CCSP 2.0, CCSbase, AllCCS, DeepCCS) on the McLean Compendium and you need reproducible, tool-shared random test splits with aggregate median MRE/RMSE metrics.

## When NOT to use

- When you only need a single train/test partition for model development rather than a multi-tool benchmark with median aggregation over 101 splits
- When the molecules of interest are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments, since the compendium training sets used here do not support them

## Inputs

- McLean Unified CCS Compendium adduct data sets from Supplemental Tables 1-3 (cleaned: 567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions)
- Isomeric SMILES string for each compendium entry (converted from neutral InChI via PubChem Identifier Exchange Service)
- Experimentally measured drift-tube CCS value for each entry
- Number of splits (101) and test fraction (30%) as split-generation parameters

## Outputs

- 101 random 70:30 calibration/test splits per adduct type, identical across all compared tools
- Per-split MRE and RMSE for each tool against McLean Compendium CCS values
- Aggregated median MRE and median RMSE per tool per adduct ([M-H]-, [M+H]+, [M+Na]+) with tool rankings

## How to apply

Load the cleaned adduct data sets from Supplemental Tables 1-3 (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions after scrubbing), retaining the isomeric SMILES for each entry. For each adduct, draw 101 random splits holding out 30% of the data set as the test set and retaining 70% for calibration/training; use a fixed random seed so the same splits are shared across all four tools being compared. For every split, submit the identical isomeric SMILES of the withheld 30% to external predictors (AllCCS and CCSbase web servers, DeepCCS command-line model) and train the local model (CCSP 2.0 linear-kernel SVR with Mordred descriptors) on the remaining 70% to predict the same withheld compounds. Compute MRE and RMSE against the experimentally measured McLean Compendium CCS values per split, then aggregate the 101 per-split metrics into median MRE and median RMSE per tool per adduct. Rationale: sharing splits across tools and aggregating medians over 101 iterations guards against single-split sampling variability, which is why published rankings (e.g., [M-H]- median MRE: CCSP 2.0 1.251%, CCSbase 1.427%, AllCCS 1.855%, DeepCCS 1.871%) should be reproduced within sampling tolerance rather than exactly.

## Related tools

- **CCSP 2.0** (Local SVR-based CCS predictor trained on the 70% calibration portion of each split and evaluated on the shared 30% test set) — https://github.com/facundof2016/CCSP2.0
- **CCSbase** (External web-server predictor receiving the identical isomeric SMILES test sets from each split)
- **AllCCS** (External web-server predictor receiving the identical isomeric SMILES test sets from each split)
- **DeepCCS** (Built-in command-line CCS model applied to the identical isomeric SMILES test sets from each split)
- **Python** (Language used to implement the split generation, descriptor calculation, and metric aggregation)

## Evaluation signals

- Reproduced [M-H]- median MRE ranking must match the published order: CCSP 2.0 (1.251%), CCSbase (1.427%), AllCCS (1.855%), DeepCCS (1.871%), within sampling variability across the 101 splits
- Reproduced median MRE values should fall near 1.251% ([M-H]-), 1.729% ([M+H]+), and 1.874% ([M+Na]+) for CCSP 2.0
- Reproduced [M-H]- median RMSE values should be near 4.876, 7.359, 9.919, and 6.068 A^2 for CCSP 2.0, CCSbase, AllCCS, and DeepCCS respectively
- Each split must contain exactly 30% of the cleaned adduct data set (e.g., ~170 of 567 [M-H]- ions held out) with no overlap between calibration and test sets
- The same 101 splits must be reused verbatim across all four tools so that per-split metrics are directly comparable

## Limitations

- Results are subject to sampling variability across the 101 random splits, so exact metric values will fluctuate between runs unless splits are fixed
- Split generation depends on scrubbed compendium data; errors in SMILES conversion from neutral InChI strings propagate to descriptor calculation and external tool submissions
- The underlying training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments

## Evidence

- [results] Load the cleaned McLean Unified CCS Compendium adduct data sets: [evidence span withheld — anchor status: not_found]
- [results] Generate 101 random 70:30 calibration/test splits per adduct: [evidence span withheld — anchor status: not_found]
