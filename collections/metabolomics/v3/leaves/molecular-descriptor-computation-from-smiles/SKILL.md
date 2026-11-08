---
name: molecular-descriptor-computation-from-smiles
description: Use when converting small-molecule SMILES from metabolomics training/test/validation
  libraries (e.g. HILIC, RPLC PlaSMA, Pathogen Box sets) into numeric descriptor features
  for RT prediction or chemical-space overlap analysis.
when_to_use_negative:
- Compounds are complex high-molecular-weight lipids (e.g. triacylglycerols, phosphatidylcholines)
  that were excluded from the Retip training landscape — descriptor-based prediction
  may be unreliable there
- The goal is retention-time adjustment for small changes in a chromatographic method
  — the study explicitly did not test this; descriptors do not capture LC buffer/ionic-strength
  interactions
- Input is already a descriptor/feature table — go directly to scaling/modeling instead
  of recomputing descriptors
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3172
tools:
- name: Retip
  role: R/Python package providing getCD() descriptor calculation, cesc()/proc.data()
    scaling, and chem.space() PCA on the descriptor matrix
  repo: https://github.com/olobion/Retip
- name: rcdk
  role: R CDK bindings used to compute the 286 2D descriptors from standardized SMILES
    (rcdk 3.4.7.1 in the paper; rcdk 3.8.1 in Retip 2.0)
  repo: https://github.com/olobion/Retip
- name: pyRetip
  role: Python package alternative for building descriptor-based RT prediction models
  repo: https://github.com/oloBion/pyRetip
- name: ggplot2
  role: Used by Retip's chem.space() to render the PCA chemical-space overlap plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.9b05765
    title: 'Retip: Retention Time Prediction for Compound Annotation in Untargeted
      Metabolomics'
  source_content_hash: sha256:738217b2dc740948b904041ccb794495d137ed5453b16eba219cefc7817cf699
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/olobion/Retip
  tools:
  - R
  - Retip
  - ggplot2
  - pyRetip
  - rcdk
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# Molecular descriptor computation from SMILES

## Summary

Computes 2D molecular descriptors (286 CDK descriptors in the Retip workflow) from standardized compound SMILES to build a quantitative structural feature matrix for downstream retention-time ML models and chemical-space PCA.

## When to use

Use when converting small-molecule SMILES from metabolomics training/test/validation libraries (e.g. HILIC, RPLC PlaSMA, Pathogen Box sets) into numeric descriptor features for RT prediction or chemical-space overlap analysis.

## When NOT to use

- Compounds are complex high-molecular-weight lipids (e.g. triacylglycerols, phosphatidylcholines) that were excluded from the Retip training landscape — descriptor-based prediction may be unreliable there
- The goal is retention-time adjustment for small changes in a chromatographic method — the study explicitly did not test this; descriptors do not capture LC buffer/ionic-strength interactions
- Input is already a descriptor/feature table — go directly to scaling/modeling instead of recomputing descriptors

## Inputs

- SMILES structures for each compound set (e.g. HILIC: 880 training, 143 test, 377 Pathogen Box validation; RPLC PlaSMA: 398 training, 96 test, 358 validation)
- Compound metadata table mapping compounds to chromatographic method (HILIC or RPLC) and set membership

## Outputs

- Centered-and-scaled 286-column 2D CDK descriptor matrix (one row per standardized compound)
- PCA projection plots of chemical space per chromatographic method (Retip chem.space(), ggplot2-based)
- Descriptor table used as ML training features for retention time models

## How to apply

Standardize structures first: remove salts and metal-containing compounds, then compute 286 2D CDK descriptors per compound using Retip's getCD() function (rcdk; the original paper used rcdk 3.4.7.1, Retip 2.0 pins rcdk 3.8.1). Drop compounds whose descriptor calculation fails rather than imputing. Center and scale descriptor columns with Retip's cesc()/proc.data() handling, which removes non-existent values and low-variance columns so descriptors are comparable across libraries. The resulting descriptor matrix feeds both ML models (random forest, BRNN, XGBoost, LightGBM, Keras, H2O autoML) and chem.space() PCA of training vs. test/validation chemical space. Note the article's caution that octanol/water partition-coefficient type descriptors were among the most important predictors, so ensure hydrophobicity-relevant descriptors are retained after filtering.

## Related tools

- **Retip** (R/Python package providing getCD() descriptor calculation, cesc()/proc.data() scaling, and chem.space() PCA on the descriptor matrix) — https://github.com/olobion/Retip
- **rcdk** (R CDK bindings used to compute the 286 2D descriptors from standardized SMILES (rcdk 3.4.7.1 in the paper; rcdk 3.8.1 in Retip 2.0)) — https://github.com/olobion/Retip
- **pyRetip** (Python package alternative for building descriptor-based RT prediction models) — https://github.com/oloBion/pyRetip
- **ggplot2** (Used by Retip's chem.space() to render the PCA chemical-space overlap plots)

## Examples

```
devtools::install_github('olobion/Retip'); desc <- Retip::getCD(HILIC_training, type='all')
```

## Evaluation signals

- Descriptor matrix yields the expected ~286 2D CDK descriptor columns per standardized compound after dropping failures and low-variance columns
- First two principal components explain ~41% of variance for HILIC and ~69% for RPLC descriptor sets (±5 percentage points tolerance)
- PCA projections show HILIC and RP training/test libraries co-distributed, with the drug-like Pathogen Box validation set slightly outside HILIC training space and Ophiorrhiza pumila RP validation co-distributed with training
- Downstream Keras RT models achieve MAE ≈0.78 min (HILIC) and ≈0.57 min (RPLC), indicating descriptors carry sufficient structural signal
- No NA/non-existent values remain after cesc()/proc.data() handling

## Limitations

- If Retip is used for predictions outside the chemical landscape of the training set, results may be more erroneous — check PCA overlap before trusting descriptors for novel compound classes
- Molecular descriptors by themselves fail to fully explain complex interactions between LC buffer systems (including ionic strengths), microenvironments, and adsorption/desorption processes; no current models yield validation MAEs better than ±1 min
- Prediction accuracy is limited by the number of compounds used for training and their chemical diversity; many chemicals in test and validation were not well represented by the training models

## Evidence

- [methods] Compute 286 2D CDK descriptors per compound with Retip's getCD() function (rcdk 3.4.7.1 in the original paper), dropping compounds that fail descriptor calculation: [evidence span withheld — anchor status: not_found]
- [methods] PCA variance explained: [evidence span withheld — anchor status: not_found]
