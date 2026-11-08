---
name: lipid-class-matched-ccs-model-training
description: Use when a metabolomics annotation workflow needs CCS-based false-positive
  filtering for lipid/lipid-like [M-H]- candidates, requiring a training set restricted
  to the same molecular superclass as the targets.
when_to_use_negative:
- Annotation targets are not lipids or lipid-like molecules — the training set must
  be customized to the user's molecular class, as CCSP requires 'the need to define
  the specific training set to be used prior to predicting CCS values'
- Adducts are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments
  — current CCSP 2.0 training sets do not support these
- User cannot work in Python/Jupyter — the article notes the 'requirement for familiarity
  with a programming language such as Python'
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: CCSP 2.0
  role: Open-source Jupyter notebook that trains the linear SVR CCS model and applies
    ΔCCS filtering with user-curated training sets
  repo: https://github.com/facundof2016/CCSP2.0
- name: Mordred V1.2.0
  role: Python package used to calculate the 1613 2D molecular descriptors from RDKit-encoded
    MOL files
- name: RDKit
  role: Encodes molecular data (MOL) files for each isomeric SMILES before descriptor
    calculation
- name: SciKit-Learn
  role: Provides the linear support vector regression model, 5-fold grid search (C,
    epsilon), and Recursive Feature Elimination
- name: PubChem (Identifier Exchange Service, PyPubChem, Biopython Entrez)
  role: Converts neutral InChI to isomeric SMILES and retrieves exact-mass candidate
    compounds within 10 ppm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:96d2c95ebe1bfa43c7e60c2e802a6c585aeeba5ab0ffc2b57bcd891d547dd9f6
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
  - PubChem (Identifier Exchange Service, PyPubChem, Biopython Entrez)
  - PubChemPy
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# lipid-class-matched-ccs-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a class-matched CCS prediction model (CCSP 2.0 linear-kernel SVR) on a lipid/lipid-like superclass training set from the McLean CCS Compendium, then use ΔCCS filtering to remove false-positive metabolite annotations at 10 ppm mass tolerance.

## When to use

Use when a metabolomics annotation workflow needs CCS-based false-positive filtering for lipid/lipid-like [M-H]- candidates, requiring a training set restricted to the same molecular superclass as the targets.

## When NOT to use

- Annotation targets are not lipids or lipid-like molecules — the training set must be customized to the user's molecular class, as CCSP requires 'the need to define the specific training set to be used prior to predicting CCS values'
- Adducts are dimers ([2M+H]+, [2M-H]-), less common adducts, or in-source fragments — current CCSP 2.0 training sets do not support these
- User cannot work in Python/Jupyter — the article notes the 'requirement for familiarity with a programming language such as Python'

## Inputs

- McLean CCS Compendium Supplemental Table 5: [M-H]- lipid and lipid-like superclass CCS measurements (129 ions for 70:30 split)
- Neutral InChI strings for each training molecule
- PubChem compound identifiers (CIDs) for mass-matched annotation candidates (43–13,984 per compound, 136,940 total)

## Outputs

- Trained linear-kernel SVR CCS prediction model on RFE-selected Mordred descriptors
- Per-candidate CCS predictions and ΔCCS filtering decisions at 2.8% threshold
- Percent of candidates filtered at 10, 7, 5, 3, 1, and 0 ppm mass windows and percent of correct annotations retained
- ROC curves and AUC scores across mass windows

## How to apply

Load [M-H]- lipid and lipid-like superclass CCS measurements from the McLean CCS Compendium (Supplemental Table 5) and perform a 70:30 split (93 calibration ions, 36 validation, 33 after excluding compounds lacking exact PubChem matches). Convert neutral InChI strings to isomeric SMILES via the PubChem Identifier Exchange Service, encode MOL files with RDKit, and compute 1613 2D descriptors with Mordred V1.2.0, discarding constant/inapplicable descriptors and Z-transforming the rest using training-set statistics. Train a linear-kernel SVR (SciKit-Learn) on the 93-ion calibration set, tuning C (0.00390625–0.25) and epsilon (0.5, 1, 5) by 5-fold grid search, then apply Recursive Feature Elimination (removing the 5 lowest-ranked descriptors per iteration until RMSECV increases) and rebuild the final model on the RFE-selected descriptors. For each validation compound, retrieve all PubChem candidates within 10 ppm exact mass, predict CCS for every candidate, and filter with a ΔCCS threshold of 2.8%.

## Related tools

- **CCSP 2.0** (Open-source Jupyter notebook that trains the linear SVR CCS model and applies ΔCCS filtering with user-curated training sets) — https://github.com/facundof2016/CCSP2.0
- **Mordred V1.2.0** (Python package used to calculate the 1613 2D molecular descriptors from RDKit-encoded MOL files)
- **RDKit** (Encodes molecular data (MOL) files for each isomeric SMILES before descriptor calculation)
- **SciKit-Learn** (Provides the linear support vector regression model, 5-fold grid search (C, epsilon), and Recursive Feature Elimination)
- **PubChem (Identifier Exchange Service, PyPubChem, Biopython Entrez)** (Converts neutral InChI to isomeric SMILES and retrieves exact-mass candidate compounds within 10 ppm)

## Examples

```
Download 'CCSP 2.0 - CCS Prediction in Jupyter.ipynb' from https://github.com/facundof2016/CCSP2.0 and open with `jupyter lab CCSP\ 2.0\ -\ CCS\ Prediction\ in\ Jupyter.ipynb`
```

## Evaluation signals

- 100% of the 33 correct annotations are retained at ΔCCS 2.8% and 10 ppm mass error
- Median percentage of candidates filtered is 36.1% at 10 ppm (19.3% of isomeric candidates at 0 ppm)
- Lipid-class ROC AUC scores fall in the range 0.748–0.776
- Model median relative prediction error for [M-H]- adducts is 1.25% against the McLean CCS Compendium
- Descriptor pipeline yields 1613 initial Mordred 2D descriptors before constant/inapplicable removal and RFE

## Limitations

- Training sets do not currently support dimers such as [2M+H]+ or [2M-H]-, or other less common adducts or in-source fragments found in ESI-MS experiments
- The specific training set must be defined prior to CCS prediction; class-matched models do not transfer across molecular classes
- Effect of experimental sources of variance on the choice of CCS threshold chosen for filtering has not been investigated
- Users need Python programming familiarity, which limits accessibility
- Google Colab version is limited to continuous operation of up to 12 hours and disconnects after ~20 minutes idle; Sklearn V1.0.2 or later is required

## Evidence

- [readme] Repository availability and notebook format: [evidence span withheld — anchor status: external_doc]
