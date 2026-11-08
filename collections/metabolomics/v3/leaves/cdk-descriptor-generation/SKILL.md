---
name: cdk-descriptor-generation
description: Use when you need to convert SMILES structures of metabolomics compound
  sets into a CDK descriptor matrix — e.g., for Retip retention-time models or PCA
  chemical-space overlap checks before training/test validation.
when_to_use_negative:
- The input is already a computed descriptor/feature matrix and the goal is model
  fitting rather than featurization
- The task is retention-time prediction itself rather than descriptor generation or
  chemical-space overlap assessment
- Compounds are outside the study's chemical classes — e.g., high-molecular-weight
  lipids such as triacylglycerols or phosphatidylcholines, which were largely excluded
  from the original training sets
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_3520
tools:
- name: Retip
  role: R package providing getCD() for 286 2D CDK descriptor calculation, cesc()/proc.data()
    for centering/scaling, and chem.space() for PCA chemical-space plots
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: CDK descriptor engine underlying getCD() (rcdk 3.4.7.1 in the original paper;
    3.8.1 in Retip 2.0)
- name: ggplot2
  role: Rendering the PCA chemical-space projection plots produced by chem.space()
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:7adb3c72dd0993847af727d843ce7d49466e7e422897833b1b84585fc1b64e04
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cdk-descriptor-generation

## Summary

Compute 2D CDK molecular descriptors (286 features) for compound sets using Retip's getCD() built on the rcdk package, then center/scale the descriptor matrix so it can be used for retention-time model building and chemical-space (PCA) overlap analysis between HILIC/RPLC training, test, and validation libraries.

## When to use

Use when you need to convert SMILES structures of metabolomics compound sets into a CDK descriptor matrix — e.g., for Retip retention-time models or PCA chemical-space overlap checks before training/test validation.

## When NOT to use

- The input is already a computed descriptor/feature matrix and the goal is model fitting rather than featurization
- The task is retention-time prediction itself rather than descriptor generation or chemical-space overlap assessment
- Compounds are outside the study's chemical classes — e.g., high-molecular-weight lipids such as triacylglycerols or phosphatidylcholines, which were largely excluded from the original training sets

## Inputs

- Compound tables with SMILES structures and set labels (train/test/validation), e.g. from MoNA/MassBank, Metabolomics Workbench, and RIKEN sources
- HILIC set (880 training, 143 test, 377 Pathogen Box validation compounds) and RPLC PlaSMA set (398 training, 96 test, 358 validation compounds)
- Salt- and metal-free, standardized structures ready for descriptor calculation

## Outputs

- A descriptor matrix of 286 2D CDK descriptor columns per compound (compounds failing calculation dropped)
- A centered-and-scaled descriptor matrix (cesc()/proc.data()) suitable for ML training or PCA
- PCA chemical-space plots (ggplot2) superimposing training, test, and validation libraries per chromatographic method
- Cumulative first-two-PC variance values (expected ~41% HILIC, ~69% RPLC)

## How to apply

Load each compound set (e.g., HILIC: 880 training, 143 test, 377 Pathogen Box validation; RPLC PlaSMA: 398 training, 96 test, 358 validation) with SMILES, standardizing structures by removing salts and metal-containing compounds. Run Retip's getCD() (rcdk 3.4.7.1 in the original paper; rcdk 3.8.1 in Retip 2.0) to compute the 286 2D CDK descriptors, dropping any compounds that fail descriptor calculation. Apply Retip's cesc()/proc.data() handling to center and scale descriptor columns, remove non-existent values, and discard low-variance columns so features are comparable across libraries. For chemical-space assessment, feed the scaled matrix into Retip's chem.space(db_rt, target, title) to superimpose training and external validation libraries in PCA space with ggplot2-based plots, then read off the cumulative variance of the first two PCs (reported: 41% HILIC, 69% RPLC).

## Related tools

- **Retip** (R package providing getCD() for 286 2D CDK descriptor calculation, cesc()/proc.data() for centering/scaling, and chem.space() for PCA chemical-space plots) — https://github.com/olobion/Retip
- **rcdk** (CDK descriptor engine underlying getCD() (rcdk 3.4.7.1 in the original paper; 3.8.1 in Retip 2.0))
- **ggplot2** (Rendering the PCA chemical-space projection plots produced by chem.space())

## Examples

```
desc <- getCD(compound_table, des_type = "extended"); desc_scaled <- cesc(desc)
```

## Evaluation signals

- Descriptor matrix has approximately 286 numeric 2D CDK descriptor columns per compound, with compounds failing calculation removed
- After cesc() centering/scaling, descriptor columns have ~zero mean and ~unit standard deviation; low-variance columns are handled by proc.data()
- First-two-PC cumulative variance reproduces reported values within ±5 percentage points: 41% for HILIC and 69% for RPLC
- PCA projections show HILIC test/validation compounds (drug-like Pathogen Box) overlapping and slightly outside the training space, while RPLC Ophiorrhiza pumila validation compounds co-distribute with training
- Structure standardization (salt removal, metal-containing compound removal) is applied before descriptor calculation

## Limitations

- If Retip is used for predictions on compounds outside the chemical landscape of the training set, results may be more erroneous — chemical-space PCA is exactly the check for this
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity
- Some compounds fail CDK descriptor calculation and are dropped; many chemicals in test and validation sets were not well represented by the training models

## Evidence

- [methods] Compute 286 2D CDK descriptors per compound with Retip's getCD() function (rcdk 3.4.7.1 in the original paper), dropping compounds that fail descriptor calculation: [evidence span withheld — anchor status: not_found]
- [results] PCA on CDK descriptors showed chemical space distributions for HILIC and RP libraries were similar, with 41% variance explained by the first two principal components for HILIC and 69% for RP.: [evidence span withheld — anchor status: not_found]
- [readme] Retention time calculation can be useful in identifying unknowns and removing false positive annotations.: "Retention time calculation can be useful in identifying unknowns and removing false positive annotations."
- [discussion] if Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous: "if Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous"
