---
name: structure-standardization-and-cleaning
description: Use when raw compound sets from sources like MoNA, MassBank, Metabolomics
  Workbench, RIKEN, or the Pathogen Box carry mixed SMILES (salts, metal-containing
  entries) that must be cleaned before computing CDK descriptors for PCA or RT modeling.
when_to_use_negative:
- The descriptor matrix is already computed, scaled, and free of salt/metal structures
  — go directly to PCA or model training
- The goal is adjusting retention predictions for small changes in chromatographic
  methods, which the authors explicitly excluded from scope
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_3172
tools:
- name: Retip
  role: Provides getCD() for CDK descriptor calculation and cesc()/proc.data() for
    handling non-existent values, low-variance columns, and centering/scaling
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: R CDK library (3.4.7.1 in the original paper; 3.8.1 in Retip 2.0) that computes
    the 286 2D molecular descriptors from standardized SMILES
  repo: https://github.com/olobion/Retip
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:49215b89193ede3bb611e4dcd6f631e61f9968f84ac586634143e4b04403971b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - R
  - Retip
  - ggplot2
  - rcdk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.96
n_claims: 25
---

# Structure standardization and cleaning

## Summary

Standardizes compound SMILES sets (removing salts and metal-containing structures) and prepares them for 2D CDK descriptor calculation in Retip, including handling non-existent descriptor values and low-variance columns prior to centering/scaling.

## When to use

Use when raw compound sets from sources like MoNA, MassBank, Metabolomics Workbench, RIKEN, or the Pathogen Box carry mixed SMILES (salts, metal-containing entries) that must be cleaned before computing CDK descriptors for PCA or RT modeling.

## When NOT to use

- The descriptor matrix is already computed, scaled, and free of salt/metal structures — go directly to PCA or model training
- The goal is adjusting retention predictions for small changes in chromatographic methods, which the authors explicitly excluded from scope

## Inputs

- Compound tables with SMILES structures from MoNA, MassBank, Metabolomics Workbench, and RIKEN PlaSMA sources
- HILIC and RPLC training/test/validation compound lists (e.g., Pathogen Box, Ophiorrhiza pumila true positives)

## Outputs

- Cleaned, salt-free, metal-free compound set with one standard SMILES per compound
- Matrix of 286 2D CDK descriptors per compound with non-existent values and low-variance columns handled
- Centered and scaled descriptor matrix ready for PCA (chem.space) or model training

## How to apply

Load the compound sets with their SMILES (e.g., HILIC: 880 training, 143 test, 377 Pathogen Box validation; RPLC PlaSMA: 398 training, 96 test, 358 validation). Standardize each structure by removing salts and metal-containing compounds, dropping entries that fail descriptor calculation. Compute 286 2D CDK descriptors per compound using Retip's getCD() function (rcdk; version 3.4.7.1 in the original paper). Apply Retip's cesc()/proc.data() handling of non-existent values and low-variance columns, then center and scale descriptor columns so downstream PCA inputs are comparable across libraries.

## Related tools

- **Retip** (Provides getCD() for CDK descriptor calculation and cesc()/proc.data() for handling non-existent values, low-variance columns, and centering/scaling) — https://github.com/olobion/Retip
- **rcdk** (R CDK library (3.4.7.1 in the original paper; 3.8.1 in Retip 2.0) that computes the 286 2D molecular descriptors from standardized SMILES) — https://github.com/olobion/Retip

## Examples

```
library(Retip); library(rcdk); mols <- parse.smiles(smiles_vec); desc <- getCD(mols); desc <- proc.data(desc); desc <- cesc(desc)
```

## Evaluation signals

- Compounds failing descriptor calculation and salt/metal-containing structures are dropped, with counts logged per library
- All 286 descriptor columns contain valid (non-NA) values after proc.data() handling; low-variance columns removed
- Descriptor columns are centered and scaled (zero mean, unit variance) before PCA
- Downstream PCA on the cleaned matrix reproduces the reported first-two-PC variance totals (41% HILIC, 69% RPLC) within ~±5 percentage points

## Limitations

- If Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity
- Many chemicals in test and validation sets were not well represented by the training models; complex high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines were excluded from the training set

## Evidence

- [methods] standardize structures by removing salts and metal-containing compounds: [evidence span withheld — anchor status: not_found]
- [methods] Center and scale descriptor columns with Retip's cesc()/proc.data() handling of non-existent values and low-variance columns: [evidence span withheld — anchor status: not_found]
