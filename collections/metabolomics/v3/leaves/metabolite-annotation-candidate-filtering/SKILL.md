---
name: metabolite-annotation-candidate-filtering
description: Use when a high-resolution MS metabolite annotation workflow yields too
  many exact-mass candidates (e.g. 43–13,984 per compound within 10 ppm) and measured
  or predictable ion-mobility CCS values are available to prune them.
when_to_use_negative:
- Adducts not covered by the training set — current CCSP 2.0 training sets do not
  support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found
  in ESI-MS experiments
- No user-curated class-matched training set can be defined prior to prediction, since
  CCSP 2.0 requires the specific training set to be chosen before CCS prediction
- Candidates lack measured or predictable CCS data (e.g. no ion-mobility dimension
  in the acquisition) — use retention-time matching or tandem MS instead
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CCSP 2.0
  role: Open-source Jupyter Notebook tool that predicts CCS values via linear SVR
    with user-curated, class-matched training sets and applies the ΔCCS filter
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred
  role: Calculates the 1613 2D molecular descriptors used as SVR input features (V1.2.0)
- name: RDKit
  role: Encodes isomeric SMILES into MOL files for descriptor calculation
- name: Biopython (Entrez)
  role: Queries PubChem for all compounds within the exact-mass tolerance of the target
    ion
- name: PubChemPy
  role: Retrieves each candidate CID for descriptor computation
- name: SciKit-Learn
  role: Trains the linear-kernel SVR with 5-fold grid-search tuning and Recursive
    Feature Elimination (requires Sklearn V1.0.2 or later)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:e63da2ff474097af49390d155cdd85da3c82b5a4de64d566d9d8ee9a9db1aabe
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
  - Biopython (Entrez)
  - CCSP 2.0
  - Jupyter Notebook
  - Mordred
  - PubChemPy
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# metabolite-annotation-candidate-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters false-positive metabolite annotation candidates from exact-mass database searches by predicting collision cross section (CCS) values with class-matched linear support vector regression (CCSP 2.0) and rejecting structures whose ΔCCS exceeds a threshold, at a defined ppm mass tolerance.

## When to use

Use when a high-resolution MS metabolite annotation workflow yields too many exact-mass candidates (e.g. 43–13,984 per compound within 10 ppm) and measured or predictable ion-mobility CCS values are available to prune them.

## When NOT to use

- Adducts not covered by the training set — current CCSP 2.0 training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments
- No user-curated class-matched training set can be defined prior to prediction, since CCSP 2.0 requires the specific training set to be chosen before CCS prediction
- Candidates lack measured or predictable CCS data (e.g. no ion-mobility dimension in the acquisition) — use retention-time matching or tandem MS instead

## Inputs

- Feature list of accurate-mass ions with adduct assignment (e.g. [M-H]-) and measured CCS values from drift-tube ion mobility MS
- Class-matched training set of experimental CCS measurements (e.g. McLean CCS Compendium Supplemental Table 5, lipid/lipid-like superclass)
- PubChem candidate lists per compound retrieved at a defined mass tolerance (exact mass within 10 ppm)
- Neutral InChI strings for training compounds and candidate structures

## Outputs

- Per-candidate predicted CCS values and ΔCCS deviations (percentage)
- Filtered candidate list (retained annotations within ΔCCS 2.8%) and percentage of candidates filtered per mass window
- Percentage of correct annotations retained per mass tolerance (10/7/5/3/1/0 ppm)
- ROC curves and area-under-curve scores per mass window

## How to apply

Query PubChem (Biopython Entrez / PubChemPy) for all compounds matching the neutral exact mass within the chosen ppm window (10, 7, 5, 3, 1, or 0 ppm). For each candidate CID, convert neutral InChI to isomeric SMILES via the PubChem Identifier Exchange Service, encode MOL files with RDKit, and compute 1613 2D descriptors with Mordred V1.2.0 (discarding constant/inapplicable descriptors and Z-transforming the rest using training-set statistics). Predict CCS with a linear-kernel SVR trained on a class-matched set from the McLean CCS Compendium (C tuned 0.00390625–0.25, epsilon in {0.5, 1, 5}, refined by Recursive Feature Elimination until RMSECV increases), then filter candidates with |ΔCCS| > 2.8%. With a class-matched [M-H]- lipid/lipid-like superclass training set, this retains 100% of correct annotations while filtering a median of 36.1% of candidates at 10 ppm.

## Related tools

- **CCSP 2.0** (Open-source Jupyter Notebook tool that predicts CCS values via linear SVR with user-curated, class-matched training sets and applies the ΔCCS filter) — https://github.com/facundof2016/CCSP2.0
- **Mordred** (Calculates the 1613 2D molecular descriptors used as SVR input features (V1.2.0))
- **RDKit** (Encodes isomeric SMILES into MOL files for descriptor calculation)
- **Biopython (Entrez)** (Queries PubChem for all compounds within the exact-mass tolerance of the target ion)
- **PubChemPy** (Retrieves each candidate CID for descriptor computation)
- **SciKit-Learn** (Trains the linear-kernel SVR with 5-fold grid-search tuning and Recursive Feature Elimination (requires Sklearn V1.0.2 or later))

## Examples

```
# In the CCSP 2.0 Jupyter notebook: load class-matched training CCS CSV, then per validation ion: Entrez esearch PubChem (10 ppm) -> PubChemPy CIDs -> RDKit MOL -> Mordred descriptors -> SVR.predict -> keep |dCCS| <= 2.8%
```

## Evaluation signals

- At a ΔCCS threshold of 2.8% and 10 ppm mass error, 100% of known correct annotations are retained
- Median percentage of candidates filtered at 10 ppm matches the class-matched benchmark of 36.1% (19.3% of isomeric candidates at 0 ppm)
- Per-compound candidate counts fall in the reported range (43–13,984 candidates per compound; 136,940 total across 33 validation compounds)
- ROC AUC scores for the lipid-class filtering fall in the reported range 0.748–0.776
- SVR median relative error on held-out McLean Compendium test ions is ≤2% (e.g. 1.25% for [M-H]-)

## Limitations

- Training sets do not support dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments found in ESI-MS experiments
- The specific training set must be defined prior to CCS prediction
- Effect of experimental sources of variance on the choice of CCS threshold not yet investigated
- CCS predictions across multiple IM-MS instrument types (drift tube, travelling wave) not yet explored
- Multi-adduct CCS filtering and CCS prediction of tandem MS fragment ions are left as future work
- Running CCSP 2.0 requires familiarity with Python, limiting accessibility; Google Colab disconnects after ~20 minutes idle and 12 hours continuous use

## Evidence

- [readme] Sklearn version requirement: [evidence span withheld — anchor status: external_doc]
