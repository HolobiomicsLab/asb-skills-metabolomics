---
name: annotation-candidate-true-positive-classification
description: Use when evaluating whether predicted-vs-measured CCS differences can
  separate true from false annotation candidates, e.g. sweeping ΔCCS thresholds across
  10, 7, 5, 3, 1, or 0 ppm mass windows to compute ROC AUC for CCSP-based filtering.
when_to_use_negative:
- When candidate lists already have validated identity assignments and no CCS prediction
  is needed — this skill only classifies candidates using predicted-vs-measured CCS
  differences
- When training and validation sets are drawn from the same instrument or chemical
  class with no diversity, so ΔCCS thresholds would be trivially small and filtering
  is not challenged
- When candidates lack a known measured CCS for the true standard — ΔCCS cannot be
  computed without it
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: Produces the predicted CCS values whose difference from measured CCS defines
    ΔCCS for true/false-positive classification
  repo: https://github.com/facundof2016/CCSP2.0
- name: PubChem
  role: Source of candidate annotation lists via the Identifier Exchange Service and
    Entrez at each mass window
- name: Python
  role: Runtime for computing ΔCCS values and sweeping thresholds; used with scikit-learn's
    roc_curve/roc_auc_score
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:3ad9d7a34199d1968cdef2bff423d2e3a7bdfe09c5b97b09e2d18968fa8687b8
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
  - Mordred
  - PubChem
  - Python
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# annotation-candidate-true-positive-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies each metabolite annotation candidate as a true or false positive by sweeping ΔCCS thresholds — the absolute relative difference between the CCSP 2.0-predicted CCS and the experimentally measured CCS of the true standard — so that ROC AUC can quantify how effectively CCS filtering removes false positives across mass-window conditions.

## When to use

Use when evaluating whether predicted-vs-measured CCS differences can separate true from false annotation candidates, e.g. sweeping ΔCCS thresholds across 10, 7, 5, 3, 1, or 0 ppm mass windows to compute ROC AUC for CCSP-based filtering.

## When NOT to use

- When candidate lists already have validated identity assignments and no CCS prediction is needed — this skill only classifies candidates using predicted-vs-measured CCS differences
- When training and validation sets are drawn from the same instrument or chemical class with no diversity, so ΔCCS thresholds would be trivially small and filtering is not challenged
- When candidates lack a known measured CCS for the true standard — ΔCCS cannot be computed without it

## Inputs

- Pre-built annotation candidate lists per mass window (10, 7, 5, 3, 1, 0 ppm) from PubChem/Entrez
- Experimentally measured CCS values for each compound (McLean Unified CCS Compendium values)
- CCSP 2.0-predicted CCS values for each candidate
- Compound identity labels distinguishing true positives from false positives
- Adduct class labels (e.g. [M-H]-, [M+H]+, [M+Na]+) and lipid superclass labels for class-matched training

## Outputs

- Per-candidate ΔCCS values (absolute relative difference between predicted and measured CCS)
- True-positive and false-positive discovery rates across the ΔCCS threshold sweep
- One ROC AUC score per mass window per data set, tabulated as results_table.csv with columns mass window (ppm), data set type, and AUC

## How to apply

For each mass-window condition, load the pre-built annotation candidate lists (generated from PubChem/Entrez at 10, 7, 5, 3, 1, and 0 ppm) for the 170-compound chemically diverse [M-H]- set and the 33-compound lipid class-matched set, together with each compound's experimentally measured CCS and its CCSP 2.0-predicted CCS. For every candidate, compute the absolute relative difference (ΔCCS) between predicted and measured CCS of the true standard, then at each ΔCCS threshold sweep label candidates as true or false positives. Feed the resulting true-positive and false-positive discovery rates across the threshold sweep into scikit-learn's roc_curve/roc_auc_score to obtain one AUC per mass window per data set. Expected reference outputs: chemically diverse set AUCs of 0.657, 0.633, 0.622, 0.616, 0.593, 0.589 and lipid class-matched set AUCs of 0.776, 0.754, 0.748, 0.768, 0.756, 0.756 for 10, 7, 5, 3, 1, 0 ppm respectively, showing near-constant AUC across mass windows for the lipid set but declining AUC with narrower mass windows for the diverse set.

## Related tools

- **CCSP 2.0** (Produces the predicted CCS values whose difference from measured CCS defines ΔCCS for true/false-positive classification) — https://github.com/facundof2016/CCSP2.0
- **PubChem** (Source of candidate annotation lists via the Identifier Exchange Service and Entrez at each mass window)
- **Python** (Runtime for computing ΔCCS values and sweeping thresholds; used with scikit-learn's roc_curve/roc_auc_score)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score
auc = roc_auc_score(y_true, y_pred)
```

## Evaluation signals

- Computed AUCs should match the reported diverse-set scores (10 ppm = 0.657; 0 ppm = 0.589) and lipid-set scores (10 ppm = 0.776; 0 ppm = 0.756) within rounding tolerance
- Reproduced table should show declining AUC with narrower mass windows for the chemically diverse set but near-constant AUC across mass windows for the lipid class-matched set
- Results table schema must contain one row per mass window per data set with columns mass window (ppm), data set type, and AUC
- ΔCCS values must be absolute relative differences — check that no candidate has a negative ΔCCS
- At ΔCCS threshold 0, the true-positive discovery rate should be 1.0 (no filtering) and the false-positive discovery rate should also be 1.0

## Limitations

- Requires a known measured CCS for the true standard — cannot classify candidates for which no experimental CCS exists
- Results depend on the training set composition; the article notes that diverse training sets show declining AUC with narrower mass windows, unlike class-matched lipid sets
- CCSP 2.0 requires familiarity with a programming language such as Python, limiting accessibility
- Current CCSP training sets do not support dimers ([2M+H]+, [2M-H]-) or less common adducts/in-source fragments, so candidates with those species cannot be classified
- The specific training set must be defined prior to CCS prediction; changing training set composition changes ΔCCS values and therefore AUC

## Evidence

- [results] AUC expected range: [evidence span withheld — anchor status: not_found]
- [results] Diverse set expected AUC decline: [evidence span withheld — anchor status: not_found]
- [methods] Classification basis: [evidence span withheld — anchor status: not_found]
- [methods] Mass windows: [evidence span withheld — anchor status: not_found]
- [results] Output table spec: [evidence span withheld — anchor status: not_found]
- [abstract] CCSP 2.0 tool role: "we report on a highly accurate machine learning algorithm (CCSP 2.0) in an open-source Jupyter Notebook format to predict CCS values"
- [abstract] Threshold filtering context: "filtering of 36.1% of incorrect structures while retaining a total of 100% of the correct annotations using a ΔCCS threshold of 2.8% and a mass error of 10 ppm"
- [intro] Measured CCS source: "All drift tube collision cross section measurements utilized for model creation and evaluation were obtained from the McLean Unified CCS Compendium24."
- [intro] Candidate generation tool: "Each molecule in the McLean database was initially represented by its neutral InChI string, which was then converted to the corresponding isomeric simplified molecular-input line-entry system"
- [discussion] Training set customization: "the need to define the specific training set to be used prior to predicting CCS values"
- [results] Diverse vs lipid qualitative pattern: [evidence span withheld — anchor status: not_found]
- [discussion] Python requirement: "These include the requirement for familiarity with a programming language such as Python"
