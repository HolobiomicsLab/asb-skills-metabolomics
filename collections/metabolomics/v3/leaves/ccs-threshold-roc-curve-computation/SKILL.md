---
name: ccs-threshold-roc-curve-computation
description: Use when evaluating annotation-candidate filtering by predicted-vs-measured
  CCS, especially when testing whether ROC AUC holds across candidate-list mass windows
  (10, 7, 5, 3, 1, 0 ppm) or across SVR training-set compositions (chemically diverse
  vs class-matched).
when_to_use_negative:
- The goal is CCS value prediction itself (CCSP 2.0 model fitting/descriptor calculation)
  rather than evaluating filtering performance — use the CCSP 2.0 prediction workflow,
  not ROC AUC evaluation
- Candidate labels are unknown (no confirmed true standard structure per compound),
  making true/false positive classification at each ΔCCS threshold impossible
- The training set involves dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source
  fragments, which the current CCSP training sets do not support
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: CCSP 2.0
  role: Supplies the predicted CCS values whose deviation from measured CCS (ΔCCS)
    defines the threshold sweep used for candidate filtering and ROC construction
  repo: https://github.com/facundof2016/CCSP2.0
- name: scikit-learn
  role: roc_curve/roc_auc_score used to compute true-positive vs false-positive discovery
    rates across the ΔCCS threshold sweep and the per-window AUC
- name: PubChem
  role: Source of annotation candidate lists via Entrez at each mass window, converted
    between neutral InChI and isomeric SMILES via the Identifier Exchange Service
- name: Mordred
  role: Calculates the 1613 2D molecular descriptors underlying the CCSP 2.0 SVR CCS
    predictions evaluated here
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:d114aad403f8e476ec60a1c657d7415ada3af87c66f75b06817eb8e7a0243d55
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
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ccs-threshold-roc-curve-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes ROC AUC scores for CCS-based filtering of metabolite annotation candidates by sweeping ΔCCS (absolute relative difference between CCSP-predicted and experimentally measured CCS) thresholds and scoring true/false positive discovery rates with scikit-learn across mass-window conditions (10 to 0 ppm). Used to quantify how well CCSP 2.0 predictions filter false positives and whether efficacy is stable under narrower precursor mass windows and different training-set compositions.

## When to use

Use when evaluating annotation-candidate filtering by predicted-vs-measured CCS, especially when testing whether ROC AUC holds across candidate-list mass windows (10, 7, 5, 3, 1, 0 ppm) or across SVR training-set compositions (chemically diverse vs class-matched).

## When NOT to use

- The goal is CCS value prediction itself (CCSP 2.0 model fitting/descriptor calculation) rather than evaluating filtering performance — use the CCSP 2.0 prediction workflow, not ROC AUC evaluation
- Candidate labels are unknown (no confirmed true standard structure per compound), making true/false positive classification at each ΔCCS threshold impossible
- The training set involves dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments, which the current CCSP training sets do not support

## Inputs

- Annotation candidate lists per compound per mass window (10, 7, 5, 3, 1, 0 ppm), generated via PubChem/Entrez
- Experimentally measured CCS values for the 170-compound chemically diverse [M-H]- validation set and the 33-compound lipid class-matched set
- CCSP-predicted CCS values for each candidate structure (CCSP 2.0, linear SVR models)
- Known true/false positive labels per candidate (or knowledge of the true standard structure) for ROC construction

## Outputs

- results_table.csv with one row per mass window per data set and columns for mass window (ppm), data set type, and AUC
- Per-mass-window ROC curves (true-positive vs false-positive discovery rate pairs from the ΔCCS threshold sweep)
- AUC comparison table: chemically diverse set (0.657, 0.633, 0.622, 0.616, 0.593, 0.589) vs lipid class-matched set (0.776, 0.754, 0.748, 0.768, 0.756, 0.756) for 10, 7, 5, 3, 1, 0 ppm

## How to apply

Load pre-built annotation candidate lists generated via PubChem/Entrez at 10, 7, 5, 3, 1 and 0 ppm mass windows together with each compound's experimentally measured CCS and the CCSP-predicted CCS value. For each mass window and candidate, classify annotation candidates as true or false positives at a sweep of ΔCCS thresholds, where ΔCCS is the absolute relative difference between predicted and measured CCS of the true standard. Compute true-positive versus false-positive discovery rates across the threshold sweep with scikit-learn's roc_curve/roc_auc_score and calculate the AUC for each of the six mass windows per data set. Rationale: the paper reports CCSP filtering removed 36.1% of incorrect structures while retaining 100% of correct annotations at a ΔCCS threshold of 2.8% and 10 ppm mass error, and ROC AUC over the threshold sweep is the chosen aggregate metric for this filtering behavior. Tabulate AUC per mass window per data set (chemically diverse 170-compound [M-H]- set vs 33-compound lipid class-matched set) to detect whether filtering efficacy declines with narrower mass windows, as it does for diverse training sets but not class-matched ones.

## Related tools

- **CCSP 2.0** (Supplies the predicted CCS values whose deviation from measured CCS (ΔCCS) defines the threshold sweep used for candidate filtering and ROC construction) — https://github.com/facundof2016/CCSP2.0
- **scikit-learn** (roc_curve/roc_auc_score used to compute true-positive vs false-positive discovery rates across the ΔCCS threshold sweep and the per-window AUC)
- **PubChem** (Source of annotation candidate lists via Entrez at each mass window, converted between neutral InChI and isomeric SMILES via the Identifier Exchange Service)
- **Mordred** (Calculates the 1613 2D molecular descriptors underlying the CCSP 2.0 SVR CCS predictions evaluated here)

## Examples

```
from sklearn.metrics import roc_curve, roc_auc_score; fpr, tpr, _ = roc_curve(y_true, d_ccs); auc = roc_auc_score(y_true, d_ccs)
```

## Evaluation signals

- Lipid class-matched set AUC values match 0.776 (10 ppm) and 0.756 (0 ppm) within rounding tolerance
- Chemically diverse set AUC values match 0.657 (10 ppm) and 0.589 (0 ppm) within rounding tolerance
- Qualitative pattern reproduced: declining AUC with narrower mass windows for the diverse set, but near-constant AUC across mass windows for the lipid class-matched set
- results_table.csv contains exactly one row per mass window (6) per data set (2) with columns mass window (ppm), data set type, and AUC
- At ΔCCS threshold 2.8% and 10 ppm mass error, filtering retains 100% of correct annotations while removing incorrect structures, consistent with the paper's reported behavior

## Limitations

- Current CCSP training sets do not support dimers such as [2M+H]+ or [2M-H]-, or other less common adducts or in-source fragments found in ESI-MS experiments
- The specific training set must be defined prior to CCS prediction, so training-set composition (diverse vs class-matched) directly conditions the resulting AUC
- The effect of experimental sources of variance on the choice of CCS threshold was not investigated in this work
- Results are anchored to drift tube CCS measurements from the McLean Unified CCS Compendium; travelling wave IM platforms are left as future work

## Evidence

- [readme] The CCSP 2.0 Jupyter notebook and its user-curated training-set capability underpin the predicted CCS values evaluated in this ROC analysis: [evidence span withheld — anchor status: external_doc]
