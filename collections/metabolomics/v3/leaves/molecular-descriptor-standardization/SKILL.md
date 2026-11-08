---
name: molecular-descriptor-standardization
description: Use when converting molecules (e.g. InChI/SMILES from the McLean CCS
  Compendium) into Mordred 2D descriptor matrices that need cleaning and scaling before
  SVR, RFECV, or other scikit-learn regression modeling.
when_to_use_negative:
- Input is already a cleaned, scaled feature table — re-pruning or re-scaling with
  new statistics would break consistency with a trained model's saved descriptor list
- Descriptors are computed for structure classes outside the training set's chemical
  space (e.g. dimers like [2M+H]+ or [2M-H]-, less common adducts, or in-source fragments),
  which CCSP 2.0's training sets do not support
- You need 3D descriptors or conformer-dependent features — this skill covers only
  the 1613 2D molecular descriptors calculated by Mordred
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0089
tools:
- name: Mordred
  role: Python package used to calculate the 1613 2D molecular descriptors that this
    skill prunes and standardizes
- name: RDKit
  role: Encodes molecular data (MOL) files for each SMILES prior to Mordred descriptor
    calculation
- name: CCSP 2.0
  role: Jupyter notebook pipeline in which descriptor pruning and Z-transformation
    precede linear SVR training and RFECV feature elimination
  repo: https://github.com/facundof2016/CCSP2.0
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:a8f4f7fcb996ec8736279e3bffc938b3fac01bf12f8a921d0cfd9efb0c2a91e4
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
  - Numpy
  - Pandas
  - Python
  - RDKit
  - SciKit-Learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# molecular-descriptor-standardization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prune raw Mordred molecular descriptors (constant values, inapplicable/division-by-zero features) and Z-transform the remaining descriptors using training-set statistics before training CCS prediction models. This prevents invalid features and scale mismatch from degrading SVR-based CCS prediction.

## When to use

Use when converting molecules (e.g. InChI/SMILES from the McLean CCS Compendium) into Mordred 2D descriptor matrices that need cleaning and scaling before SVR, RFECV, or other scikit-learn regression modeling.

## When NOT to use

- Input is already a cleaned, scaled feature table — re-pruning or re-scaling with new statistics would break consistency with a trained model's saved descriptor list
- Descriptors are computed for structure classes outside the training set's chemical space (e.g. dimers like [2M+H]+ or [2M-H]-, less common adducts, or in-source fragments), which CCSP 2.0's training sets do not support
- You need 3D descriptors or conformer-dependent features — this skill covers only the 1613 2D molecular descriptors calculated by Mordred

## Inputs

- Molecules as neutral InChI strings or isomeric SMILES (e.g. from the McLean Unified CCS Compendium)
- RDKit-encoded MOL files per SMILES
- Raw 1613-column Mordred V1.2.0 2D descriptor matrix
- Adduct-defined calibration sets (e.g. 567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions)

## Outputs

- Pruned descriptor matrix with constant and inapplicable descriptors removed
- Z-transformed descriptor table using training-set mean and standard deviation
- Validation-set descriptors with >1000 SD outliers excluded
- Train/validation split (70%/30%) aligned with the cleaned descriptor matrix

## How to apply

First convert neutral InChI strings to isomeric SMILES via the PubChem Identifier Exchange Service, encode MOL files with RDKit, and calculate 1613 2D molecular descriptors with Mordred V1.2.0. Discard descriptors that are constant across calibration molecules or inapplicable (e.g. division-by-zero bond-path descriptors). Randomly split molecules into 70% training and 30% external validation, then Z-transform the remaining descriptors using the training-set mean and standard deviation only. Exclude any test datapoint exceeding 1000 standard deviations from its training-set mean to avoid extrapolation artifacts before fitting linear-kernel SVR models.

## Related tools

- **Mordred** (Python package used to calculate the 1613 2D molecular descriptors that this skill prunes and standardizes)
- **RDKit** (Encodes molecular data (MOL) files for each SMILES prior to Mordred descriptor calculation)
- **CCSP 2.0** (Jupyter notebook pipeline in which descriptor pruning and Z-transformation precede linear SVR training and RFECV feature elimination) — https://github.com/facundof2016/CCSP2.0

## Examples

```
from mordred import Calculator, descriptors; from rdkit import Chem; calc = Calculator(descriptors); X = calc.pandas([Chem.MolFromMolFile(m) for m in molfiles])
```

## Evaluation signals

- No descriptor column retained has a constant (zero-variance) value across the calibration molecules
- Descriptor scaling was fit on the 70% training split only; validation-set transform must reuse the training-set mean and standard deviation
- No retained validation datapoint lies more than 1000 standard deviations from its descriptor mean
- Downstream per-adduct SVR median relative errors on the 30% hold-out set reach ~2% or below (paper reports 1.25%, 1.73% and 1.87% for [M-H]-, [M+H]+ and [M+Na]+)
- Descriptor count after pruning matches the RFECV-selected subsets (~370, 303 and 414 descriptors across the three adduct types)

## Limitations

- Descriptors deemed inapplicable (e.g. division-by-zero bond-path descriptors) must be removed per adduct dataset; blindly keeping them breaks model fitting
- Z-transformation statistics are training-set-specific; a customized user training set requires recomputing means/standard deviations before prediction
- The sklearn version matters: CCSP 2.0 requires scikit-learn V1.0.2 or later (see requirements.txt in the repository)

## Evidence

- [methods] Discard descriptors that are constant across calibration molecules or inapplicable (e.g. division-by-zero bond-path descriptors): [evidence span withheld — anchor status: not_found]
- [methods] Z-transform remaining descriptors using training-set mean and standard deviation, excluding any test datapoint exceeding 1000 standard deviations from its mean: [evidence span withheld — anchor status: not_found]
- [readme] The current version of CCSP 2.0 requires Sklearn V1.0.2 or later.: [evidence span withheld — anchor status: external_doc]
