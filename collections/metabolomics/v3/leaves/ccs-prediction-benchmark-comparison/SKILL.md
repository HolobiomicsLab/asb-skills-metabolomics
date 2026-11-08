---
name: ccs-prediction-benchmark-comparison
description: Use when comparing the prediction accuracy of multiple CCS prediction
  tools on the same test sets, or when reproducing reported median MRE/RMSE rankings
  for [M-H]-, [M+H]+, or [M+Na]+ adducts.
when_to_use_negative:
- The task is predicting CCS values for new compounds rather than benchmarking tool
  accuracy — use the individual prediction tools (e.g. CCSP 2.0 notebook) directly
  instead
- The compounds are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments
  — the CCSP 2.0 training sets do not support these, so a fair benchmark cannot be
  run
- Only a single tool's accuracy on one fixed train/test split is needed — the 101-split
  median aggregation is designed for multi-tool ranking, not single-tool validation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CCSP 2.0
  role: Tool under benchmark; linear-kernel SVR models trained per split on 70% calibration
    data with Mordred descriptors (grid-searched C and epsilon, 5-fold CV, RFE selection)
    to predict the withheld 30%
  repo: https://github.com/facundof2016/CCSP2.0
- name: CCSbase
  role: Compared machine-learning CCS prediction method; queried with the identical
    isomeric SMILES for each test set
- name: AllCCS
  role: Compared machine-learning CCS prediction method; queried with the identical
    isomeric SMILES for each test set via its web server
- name: DeepCCS
  role: Compared machine-learning CCS prediction method; run with its built-in command-line
    model on the shared test sets
- name: Mordred V1.2.0
  role: Calculates 1613 2D molecular descriptors from RDKit-encoded MOL files as input
    features for the CCSP 2.0 SVR models
- name: RDKit
  role: Encodes molecular data (MOL) files for each isomeric SMILES prior to descriptor
    calculation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:84738895b5168832e6418e8a59ff5fa8e3a07e889586c81fa6e81d264f54475f
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
  - CCSBase
  - CCSP 2.0
  - CCSbase
  - DeepCCS
  - Mordred
  - Mordred V1.2.0
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- ccs-prediction-benchmarking
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ccs-prediction-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks CCS prediction tools (CCSP 2.0, CCSbase, AllCCS, DeepCCS) by computing per-split MRE and RMSE against experimentally measured McLean Compendium CCS values across 101 random 70:30 calibration/test splits, then ranking tools by median metrics per adduct.

## When to use

Use when comparing the prediction accuracy of multiple CCS prediction tools on the same test sets, or when reproducing reported median MRE/RMSE rankings for [M-H]-, [M+H]+, or [M+Na]+ adducts.

## When NOT to use

- The task is predicting CCS values for new compounds rather than benchmarking tool accuracy — use the individual prediction tools (e.g. CCSP 2.0 notebook) directly instead
- The compounds are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments — the CCSP 2.0 training sets do not support these, so a fair benchmark cannot be run
- Only a single tool's accuracy on one fixed train/test split is needed — the 101-split median aggregation is designed for multi-tool ranking, not single-tool validation

## Inputs

- Cleaned McLean Unified CCS Compendium adduct data sets from Supplemental Tables 1-3 (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions) with isomeric SMILES per entry
- Isomeric SMILES strings for each test-set molecule (obtained by converting neutral InChI strings via the PubChem Identifier Exchange Service)
- Experimentally measured drift-tube CCS values from the McLean Compendium for each test ion

## Outputs

- Per-split MRE and RMSE for each of the four tools across 101 splits and three adduct types
- Median MRE and median RMSE per tool per adduct (e.g. [M-H]- MRE: CCSP 2.0 1.251%, CCSbase 1.427%, AllCCS 1.855%, DeepCCS 1.871%)
- Ranking of the four CCS prediction tools per adduct by median MRE and median RMSE

## How to apply

Load cleaned McLean Unified CCS Compendium adduct data sets (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions) with isomeric SMILES, then generate 101 random 70:30 calibration/test splits per adduct, keeping the identical splits shared across all four tools. Submit the same isomeric SMILES for each 30% test set to the AllCCS and CCSbase web servers and the DeepCCS command-line model, while training CCSP 2.0's linear-kernel SVR (Mordred descriptors, grid-searched C and epsilon with 5-fold cross-validation, RFE descriptor selection) on the remaining 70%. Compute MRE and RMSE against the experimental McLean Compendium CCS values for each of the 101 splits and all three adducts, aggregate into median MRE and median RMSE per tool per adduct, and rank the tools. Reference reproduced values: for [M-H]-, median MRE of 1.251% (CCSP 2.0), 1.427% (CCSbase), 1.855% (AllCCS), 1.871% (DeepCCS) and RMSE of 4.876, 7.359, 9.919, 6.068 A^2 respectively; for CCSP 2.0, MRE of 1.251%, 1.729%, 1.874% and RMSE of 4.876, 5.755, 6.556 A^2 for [M-H]-, [M+H]+, [M+Na]+.

## Related tools

- **CCSP 2.0** (Tool under benchmark; linear-kernel SVR models trained per split on 70% calibration data with Mordred descriptors (grid-searched C and epsilon, 5-fold CV, RFE selection) to predict the withheld 30%) — https://github.com/facundof2016/CCSP2.0
- **CCSbase** (Compared machine-learning CCS prediction method; queried with the identical isomeric SMILES for each test set)
- **AllCCS** (Compared machine-learning CCS prediction method; queried with the identical isomeric SMILES for each test set via its web server)
- **DeepCCS** (Compared machine-learning CCS prediction method; run with its built-in command-line model on the shared test sets)
- **Mordred V1.2.0** (Calculates 1613 2D molecular descriptors from RDKit-encoded MOL files as input features for the CCSP 2.0 SVR models)
- **RDKit** (Encodes molecular data (MOL) files for each isomeric SMILES prior to descriptor calculation)

## Examples

```
jupyter lab "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"  # train SVR on 70% split, predict 30% test set, then compute MRE/RMSE vs McLean Compendium CCS
```

## Evaluation signals

- Reproduced [M-H]- median MRE ranking matches the published order CCSP 2.0 (1.251%) > CCSbase (1.427%) > AllCCS (1.855%) > DeepCCS (1.871%), within sampling variability across the 101 splits
- Reproduced [M-H]- median RMSE values are approximately 4.876, 7.359, 9.919, and 6.068 A^2 for CCSP 2.0, CCSbase, AllCCS, and DeepCCS respectively
- CCSP 2.0 median MRE falls near 1.251%, 1.729%, and 1.874% and median RMSE near 4.876, 5.755, and 6.556 A^2 for [M-H]-, [M+H]+, and [M+Na]+ respectively
- All four tools are evaluated on the identical 30% test molecules per split — verify split consistency across tools before computing metrics
- Per-split metrics exist for all 101 splits and all three adduct types before median aggregation

## Limitations

- CCSP 2.0 training sets do not support dimers such as [2M+H]+ or [2M-H]-, or other less common adducts or in-source fragments found in ESI-MS experiments
- The specific training set must be defined prior to CCS prediction when using CCSP 2.0
- CCS predictions across multiple IM-MS instrument types (drift tube, travelling wave) were not explored
- Use of CCSP 2.0 requires familiarity with a programming language such as Python
- CCSbase and AllCCS web-server submission and DeepCCS command-line execution can be slower/less reproducible than the local open-source notebook approach

## Evidence
