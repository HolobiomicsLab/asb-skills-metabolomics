---
name: mass-window-candidate-list-generation
description: Use when building annotation candidate lists at multiple parent-mass
  windows (down to 0 ppm isobaric) to test whether CCS filtering of false positives
  remains stable as mass discrimination is tightened.
when_to_use_negative:
- You only need a single broad mass tolerance for one-shot annotation filtering rather
  than a comparison across windows — a single 10 ppm ΔCCS threshold filter (2.8%)
  suffices
- Your candidates involve dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source
  fragments, which the CCSP training sets do not support
- The training set needed for the CCS prediction cannot be defined prior to prediction
  (CCSP requires a specific training set to be chosen upfront)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: Predicts CCS values for each candidate structure so candidates can be scored
    by ΔCCS at each mass window
  repo: https://github.com/facundof2016/CCSP2.0
- name: PubChem
  role: Supplies candidate structures via identifier conversion (neutral InChI to
    isomeric SMILES) and candidate retrieval for mass windows
- name: RDKit
  role: Encodes molecular data (MOL) files for each candidate SMILES prior to descriptor
    calculation
- name: Mordred
  role: Calculates the 1613 2D molecular descriptors from each MOL file used as SVR
    features
- name: Python
  role: Environment in which candidate-list processing, ΔCCS classification, and ROC
    AUC computation are performed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:595010369ae653dc64524c3b87389d7476932ce7ee50a513340bf23133d30786
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
  - RDKit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mass-window-candidate-list-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates metabolite annotation candidate lists at successively narrower parent-mass tolerances (10, 7, 5, 3, 1, and 0 ppm isobaric) from PubChem/Entrez, then pairs each candidate with CCSP-predicted CCS values so that CCS-based filtering efficacy can be evaluated as a function of mass-window width.

## When to use

Use when building annotation candidate lists at multiple parent-mass windows (down to 0 ppm isobaric) to test whether CCS filtering of false positives remains stable as mass discrimination is tightened.

## When NOT to use

- You only need a single broad mass tolerance for one-shot annotation filtering rather than a comparison across windows — a single 10 ppm ΔCCS threshold filter (2.8%) suffices
- Your candidates involve dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments, which the CCSP training sets do not support
- The training set needed for the CCS prediction cannot be defined prior to prediction (CCSP requires a specific training set to be chosen upfront)

## Inputs

- Observed parent m/z values for a validation set (e.g. 170-compound chemically diverse [M-H]- set, 33-compound lipid class-matched set)
- Experimentally measured CCS values for each standard
- Candidate molecular structures retrieved from PubChem/Entrez at each mass window
- Neutral InChI strings convertible to isomeric SMILES via PubChem Identifier Exchange
- A CCSP 2.0 training set (chemically diverse or class-matched) chosen before prediction

## Outputs

- Candidate annotation lists per compound per mass window (10, 7, 5, 3, 1, 0 ppm)
- CCSP-predicted CCS values for every candidate
- True/false positive labels across a sweep of ΔCCS thresholds
- Per-window ROC AUC scores from scikit-learn roc_curve/roc_auc_score
- results_table.csv with columns: mass window (ppm), data set type, AUC

## How to apply

For each observed parent mass, retrieve formula/candidate structures from PubChem/Entrez at six mass-error windows (10, 7, 5, 3, 1, and 0 ppm, where 0 ppm means isobaric candidates only), building a separate candidate list per window for each validation set (e.g. the 170-compound chemically diverse [M-H]- set and the 33-compound lipid class-matched set). Convert each candidate's neutral InChI string to isomeric SMILES using the PubChem Identifier Exchange Service, encode MOL files with RDKit, and calculate the 1613 2D molecular descriptors with Mordred V1.2.0. Predict CCS for every candidate with CCSP 2.0's linear support vector regression model, using a training set matched to the evaluation goal (chemically diverse vs. lipid-superclass-matched). For each mass window, classify candidates as true or false positives against the known true standard at a sweep of ΔCCS thresholds based on the absolute relative difference between predicted and measured CCS, then compute true-positive versus false-positive discovery rates and the AUC with scikit-learn's roc_curve/roc_auc_score. Tabulate one AUC per mass window per data set into results_table.csv with columns for mass window (ppm), data set type, and AUC, so the mass-window dependence of filtering can be compared directly.

## Related tools

- **CCSP 2.0** (Predicts CCS values for each candidate structure so candidates can be scored by ΔCCS at each mass window) — https://github.com/facundof2016/CCSP2.0
- **PubChem** (Supplies candidate structures via identifier conversion (neutral InChI to isomeric SMILES) and candidate retrieval for mass windows)
- **RDKit** (Encodes molecular data (MOL) files for each candidate SMILES prior to descriptor calculation)
- **Mordred** (Calculates the 1613 2D molecular descriptors from each MOL file used as SVR features)
- **Python** (Environment in which candidate-list processing, ΔCCS classification, and ROC AUC computation are performed)

## Examples

```
for ppm in [10, 7, 5, 3, 1, 0]: lo, hi = obs_mz * (1 - ppm * 1e-6), obs_mz * (1 + ppm * 1e-6)
```

## Evaluation signals

- Chemically diverse set AUCs match the expected values 0.657, 0.633, 0.622, 0.616, 0.593, 0.589 for 10, 7, 5, 3, 1, 0 ppm within rounding tolerance (spot-check 10 ppm = 0.657; 0 ppm = 0.589)
- Lipid class-matched set AUCs match 0.776, 0.754, 0.748, 0.768, 0.756, 0.756 within rounding tolerance (spot-check 10 ppm = 0.776; 0 ppm = 0.756)
- Qualitative pattern reproduced: declining AUC with narrower mass windows for the diverse set, but near-constant AUC across windows for the lipid class-matched set
- results_table.csv schema check: one row per mass window per data set with columns mass window (ppm), data set type, and AUC
- Each candidate list at 0 ppm contains only isobaric candidates, and six lists exist per compound across the window sweep

## Limitations

- With a chemically diverse SVR training set, filtering efficacy declines as the mass window narrows (AUC drops from 0.657 at 10 ppm to 0.589 at 0 ppm), so diverse-set filtering degrades toward isobaric conditions
- The specific training set must be defined prior to CCS prediction; class-matched (lipid superclass) training sets are required to keep AUC stable across windows
- Current CCSP training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments
- Using CCSP 2.0 requires familiarity with Python, and the local Jupyter route requires Sklearn V1.0.2 or later

## Evidence

- [readme] CCSP purpose: [evidence span withheld — anchor status: external_doc]
