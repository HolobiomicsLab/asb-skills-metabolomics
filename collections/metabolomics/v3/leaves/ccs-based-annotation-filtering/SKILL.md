---
name: ccs-based-annotation-filtering
description: Use when annotating IM-MS features in untargeted metabolomics and a list
  of molecular-formula/structure candidates exists for an m/z within ~10 ppm, and
  an experimental CCS value is available to prune false positives.
when_to_use_negative:
- No experimental CCS value is available for the target feature, so no ΔCCS window
  can be defined
- The annotation involves dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source
  fragments, which the current CCSP 2.0 training sets do not support
- Candidates are already restricted to a verified standard or structure-confirmed
  annotation where CCS filtering adds no discriminating power
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CCSP 2.0
  role: Linear-kernel SVR CCS prediction from Mordred descriptors used to score and
    filter annotation candidates
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred V1.2.0
  role: Calculates the 1613 2D molecular descriptors that serve as SVR input features
- name: RDKit
  role: Encodes isomeric SMILES into MOL files prior to descriptor calculation
- name: PubChem
  role: Source of candidate structures retrieved via the Entrez Global Query Cross-Database
    Search System within the mass-error window
- name: Jupyter Notebook
  role: Execution environment for the CCSP 2.0 prediction and filtering workflow (Google
    Colab or local Jupyter Lab)
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:b610e2c28f3fd7695024d95c5dfe6b27acee2a32c4e58dba447d45a343e9c5c6
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
  - PubChem
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# ccs-based-annotation-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters false-positive metabolite annotation candidates by predicting collision cross section (CCS) values with CCSP 2.0 linear-kernel SVR models and retaining only candidates whose predicted CCS falls within a ΔCCS threshold of the experimental value.

## When to use

Use when annotating IM-MS features in untargeted metabolomics and a list of molecular-formula/structure candidates exists for an m/z within ~10 ppm, and an experimental CCS value is available to prune false positives.

## When NOT to use

- No experimental CCS value is available for the target feature, so no ΔCCS window can be defined
- The annotation involves dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments, which the current CCSP 2.0 training sets do not support
- Candidates are already restricted to a verified standard or structure-confirmed annotation where CCS filtering adds no discriminating power

## Inputs

- Experimental CCS value (Å2) measured by drift-tube ion mobility for the annotation target
- Adduct type (e.g., [M-H]-, [M+H]+, [M+Na]+) and m/z with a mass-error tolerance (10 ppm)
- Candidate structure list with isomeric SMILES (e.g., PubChem CIDs within the mass window)
- CCSP 2.0 SVR model trained on the relevant adduct subset of the McLean Unified CCS Compendium

## Outputs

- Filtered candidate list retaining only structures with predicted CCS within the ΔCCS threshold of the experimental value
- Per-candidate predicted CCS values and ΔCCS percentages (e.g., written to filtering_stats.csv in the threshold-sweep workflow)

## How to apply

Retrieve candidate structures (e.g., from PubChem via the Entrez Global Query Cross-Database Search System with Biopython, within a 10 ppm exact-mass window) and obtain isomeric SMILES per CID with PubChemPy. Encode each SMILES as a MOL file with RDKit, compute 1613 2D molecular descriptors with Mordred V1.2.0, and predict CCS for every candidate using the CCSP 2.0 linear-kernel SVR model trained on the appropriate adduct subset (e.g., [M-H]-) of the McLean Unified CCS Compendium. Retain only candidates whose predicted CCS lies within a ΔCCS window (e.g., ±2% for a single annotation such as isoorotic acid at 120.22 Å2, or ~2.8–3% for cohort-level filtering) of the experimental CCS. Validate the run by comparing retained/filtered counts to expected statistics (e.g., 32 of 221 candidates retained at ±2%, or 76.5% true annotations retained with 47.3% median candidates filtered at ΔCCS 3%).

## Related tools

- **CCSP 2.0** (Linear-kernel SVR CCS prediction from Mordred descriptors used to score and filter annotation candidates) — https://github.com/facundof2016/CCSP2.0
- **Mordred V1.2.0** (Calculates the 1613 2D molecular descriptors that serve as SVR input features)
- **RDKit** (Encodes isomeric SMILES into MOL files prior to descriptor calculation)
- **PubChem** (Source of candidate structures retrieved via the Entrez Global Query Cross-Database Search System within the mass-error window)
- **Jupyter Notebook** (Execution environment for the CCSP 2.0 prediction and filtering workflow (Google Colab or local Jupyter Lab)) — https://github.com/facundof2016/CCSP2.0

## Examples

```
jupyter notebook "CCSP 2.0 - CCS Prediction in Jupyter.ipynb"  # predict CCS per candidate, then keep candidates with abs(pred - exp)/exp <= 0.02
```

## Evaluation signals

- For the isoorotic acid case study: exactly 221 PubChem candidates returned at 10 ppm for m/z 155.0093, and exactly 32 retained at the ±2% ΔCCS threshold (86% of false positives removed) with the correct annotation (predicted CCS 119.3 vs experimental 120.22 Å2) kept
- For the 162-compound validation: at ΔCCS 3% and 10 ppm, 76.5% of true annotations retained, median 47.3% of candidates filtered (24.9% for same-formula candidates), over 598,208 total candidates
- Predicted CCS median relative errors should fall near 1.25% for [M-H]-, 1.73% for [M+H]+, and 1.87% for [M+Na]+ against the McLean CCS Compendium
- The correct annotation must be retained at the chosen threshold (article reports 100% of correct annotations retained at ΔCCS 2.8%)
- Filtering_stats.csv threshold sweep must show the expected monotonic trade-off between true-annotation retention and candidates filtered across 0–15% thresholds

## Limitations

- Current training sets do not support dimers such as [2M+H]+ or [2M-H]-, other less common adducts, or in-source fragments found in ESI-MS experiments
- Use of CCSP 2.0 requires familiarity with Python, limiting accessibility
- The specific training set must be defined prior to CCS prediction
- Effect of experimental sources of variance on the choice of CCS threshold not yet investigated; CCS predictions across drift tube vs travelling wave IM platforms not yet explored

## Evidence

- [readme] README on training-set customization: [evidence span withheld — anchor status: external_doc]
