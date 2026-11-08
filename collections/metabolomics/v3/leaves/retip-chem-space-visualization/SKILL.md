---
name: retip-chem-space-visualization
description: Use when you have SMILES-annotated training/test/validation compound
  sets for a chromatographic method and need to assess structural coverage before
  trusting RT-prediction generalization.
when_to_use_negative:
- Compounds are not yet standardized — salts or metal-containing structures present
  will fail or distort CDK descriptor calculation and must be removed first
- The goal is quantitative prediction-error benchmarking (MAE, cross-validation) rather
  than exploratory structural-overlap assessment — use Retip's model-building workflow
  instead
- The compound set lacks SMILES structures or the descriptor matrix cannot be computed
  (e.g., non-small-molecule entries like high molecular weight lipids excluded in
  the original training)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Retip
  role: Provides getCD(), cesc()/proc.data(), and chem.space(db_rt, target, title)
    functions used to compute descriptors, scale them, and run/plot PCA chemical space
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: R interface to the CDK descriptor engine computing the 286 2D molecular descriptors
    (version 3.4.7.1 in the paper; 3.8.1 in Retip 2.0 install block)
  repo: https://github.com/olobion/Retip
- name: ggplot2
  role: Graphics backend for the PCA chemical-space projection plots produced by chem.space()
  repo: https://github.com/olobion/Retip
- name: R
  role: Runtime environment for the Retip package workflow (R 4.4.0 required for Retip
    2.0)
  repo: https://github.com/olobion/Retip
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:da0eaec050f19f6c01958dbf3b95e03e7fd3a86543c55313d0dd75a9a0e97096
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - R
  - Retip
  - ggplot2
  - rcdk
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# retip-chem-space-visualization

## Summary

Visualizes chemical-space overlap between training, test, and validation compound libraries by running PCA on 286 2D CDK descriptors with Retip's chem.space() function, to judge whether an RT prediction model can generalize beyond its training set.

## When to use

Use when you have SMILES-annotated training/test/validation compound sets for a chromatographic method and need to assess structural coverage before trusting RT-prediction generalization.

## When NOT to use

- Compounds are not yet standardized — salts or metal-containing structures present will fail or distort CDK descriptor calculation and must be removed first
- The goal is quantitative prediction-error benchmarking (MAE, cross-validation) rather than exploratory structural-overlap assessment — use Retip's model-building workflow instead
- The compound set lacks SMILES structures or the descriptor matrix cannot be computed (e.g., non-small-molecule entries like high molecular weight lipids excluded in the original training)

## Inputs

- SMILES-annotated retention-time database table (e.g., HILIC MoNA/MassBank and metabolomicsworkbench sets; RPLC RIKEN PlaSMA sets) with train/test/validation split labels
- Per-compound structures standardized to remove salts and metal-containing compounds

## Outputs

- ggplot2-based PCA projection plots superimposing training and test/validation libraries in chemical space, one per chromatographic method
- Cumulative proportion of variance explained by the first two principal components per method (expected ~41% for HILIC, ~69% for RPLC)
- Qualitative assessment of train/external-set structural overlap (e.g., validation set slightly outside HILIC training space; RP validation co-distributed with training)

## How to apply

Load the compound sets (e.g., HILIC: 880 training, 143 test, 377 Pathogen Box validation; RPLC PlaSMA: 398 training, 96 test, 358 validation) with their SMILES and standardize structures by removing salts and metal-containing compounds. Compute 286 2D CDK descriptors per compound using Retip's getCD() function (rcdk; 3.4.7.1 in the original paper), dropping compounds that fail descriptor calculation. Center and scale descriptor columns with Retip's cesc()/proc.data() handling of non-existent values and low-variance columns so PCA inputs are comparable across libraries. Run PCA on each method's combined descriptor matrix via Retip's chem.space(db_rt, target, title), which produces ggplot2-based plots superimposing training and external validation libraries in PCA space. Record the cumulative variance explained by the first two PCs per method and visually inspect whether validation compounds (e.g., drug-like Pathogen Box for HILIC) fall inside or outside the training chemical landscape.

## Related tools

- **Retip** (Provides getCD(), cesc()/proc.data(), and chem.space(db_rt, target, title) functions used to compute descriptors, scale them, and run/plot PCA chemical space) — https://github.com/olobion/Retip
- **rcdk** (R interface to the CDK descriptor engine computing the 286 2D molecular descriptors (version 3.4.7.1 in the paper; 3.8.1 in Retip 2.0 install block)) — https://github.com/olobion/Retip
- **ggplot2** (Graphics backend for the PCA chemical-space projection plots produced by chem.space()) — https://github.com/olobion/Retip
- **R** (Runtime environment for the Retip package workflow (R 4.4.0 required for Retip 2.0)) — https://github.com/olobion/Retip

## Examples

```
devtools::install_github('olobion/Retip'); chem.space(db_rt, target, title)
```

## Evaluation signals

- First two principal components explain ~41% of variance for HILIC and ~69% for RPLC (tolerance ±5 percentage points vs. reported values)
- PCA projection plots visually reproduce the reported overlap pattern: HILIC validation set (Pathogen Box) slightly outside the training space; RPLC Ophiorrhiza pumila validation co-distributed with training
- Compounds failing CDK descriptor calculation and non-existent/low-variance descriptor columns are handled (dropped/processed) before PCA, per getCD()/cesc() behavior
- Descriptor columns are centered and scaled so all 286 CDK descriptors contribute comparably across the HILIC and RP libraries

## Limitations

- If Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous — this check is qualitative, not a guarantee of generalization
- Prediction accuracy is generally limited by the number of compounds used for training and their chemical diversity; many chemicals in test and validation were not well represented by the training models
- PCA of 2D descriptors may not fully capture LC-specific interactions (buffer ionic strength, microenvironments, adsorption/desorption), so descriptor-based chemical space is an incomplete proxy for retention behavior

## Evidence

- [results] PCA on CDK descriptors showed chemical space distributions for HILIC and RP libraries were similar, with 41% variance explained by the first two principal components for HILIC and 69% for RP.: [evidence span withheld — anchor status: not_found]
- [methods] Compute 286 2D CDK descriptors per compound with Retip's getCD() function (rcdk 3.4.7.1 in the original paper): [evidence span withheld — anchor status: not_found]
- [methods] Run principal component analysis on each chromatographic method's combined descriptor matrix using Retip's chem.space(db_rt, target, title) to superimpose training and test/validation libraries in PCA space (ggplot2-based plots).: [evidence span withheld — anchor status: not_found]
