---
name: molecular-descriptor-generation-from-smiles
description: Use when a set of small-molecule structures is given as SMILES or InChIKey
  strings and a computable numeric feature matrix is needed for retention-time regression
  or property prediction of xenobiotic metabolites.
when_to_use_negative:
- Input is already a numeric feature table — descriptor computation is redundant
- Structures lack SMILES/InChIKey strings (e.g., only names or m/z features without
  candidate structures), so descriptors cannot be computed
- The goal is MS2 spectral matching against a reference library rather than in silico
  RT prediction without authentic standards
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CMDN
  role: Supplies the reaction-guided annotation of pesticide metabolites whose SMILES/InChIKey
    structures become the input for descriptor-based RT prediction
  repo: https://github.com/LeaveMeNotTonight/CMDN
- name: LC-HRMS
  role: Provides the experimentally measured retention times that form the regression
    target paired with the computed descriptors
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:befab4c93f7b23ad3502d57438dfadec5af7cbd9244379d9515b3b0948a810ac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - CMDN
  - LC-HRMS
  - scikit-learn Random Forest
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Molecular Descriptor Generation from SMILES

## Summary

Converts chemical structures (SMILES/InChIKey) of the 1,021-pesticide library and CMDN-annotated metabolites into a molecular-descriptor feature matrix (e.g., RDKit descriptors) so that measured LC-HRMS retention times can be regressed against computable structural properties.

## When to use

Use when a set of small-molecule structures is given as SMILES or InChIKey strings and a computable numeric feature matrix is needed for retention-time regression or property prediction of xenobiotic metabolites.

## When NOT to use

- Input is already a numeric feature table — descriptor computation is redundant
- Structures lack SMILES/InChIKey strings (e.g., only names or m/z features without candidate structures), so descriptors cannot be computed
- The goal is MS2 spectral matching against a reference library rather than in silico RT prediction without authentic standards

## Inputs

- Pesticide/metabolite library with SMILES or InChIKey structure strings (1,021 pesticides; 2,886 CMDN-annotated biotransformed derivatives)
- Experimentally measured LC-HRMS retention times (minutes) for the training compounds

## Outputs

- Molecular descriptor feature matrix (one row per compound, descriptors from SMILES/InChIKey)
- Descriptor-RT dataset ready for 80:20 stratified train/test splitting and regression model training
- rt_prediction_metrics.csv with RMSE, R2, MAE per model (XGBoost: RMSE 0.89 min, R2 = 0.79, MAE = 0.60 min; 95.1% of XGB errors within 1 min)

## How to apply

Load the 1,021-pesticide library annotated with SMILES/InChIKey structures together with experimentally measured LC-HRMS retention times for the training compounds. Compute molecular descriptors from the structure strings (e.g., RDKit descriptors) to build the feature matrix for RT regression. Partition the descriptor-RT dataset into an 80:20 training/test split, stratified on the RT distribution, before fitting regressors (XGBoost, Random Forest, BRNN). The rationale is that when authentic standards are unavailable for CMDN-annotated metabolites, descriptor-based in silico RT prediction provides an orthogonal validation channel for chromatographic behavior. Write the resulting metrics (RMSE, R2, MAE) to rt_prediction_metrics.csv for comparison against reported values.

## Related tools

- **CMDN** (Supplies the reaction-guided annotation of pesticide metabolites whose SMILES/InChIKey structures become the input for descriptor-based RT prediction) — https://github.com/LeaveMeNotTonight/CMDN
- **LC-HRMS** (Provides the experimentally measured retention times that form the regression target paired with the computed descriptors)

## Examples

```
python -c "from rdkit import Chem; from rdkit.Chem import Descriptors; m=Chem.MolFromSmiles('CC(=O)Oc1ccccc1C(=O)O'); d={n:f(m) for n,f in Descriptors._descList}; print(d)"
```

## Evaluation signals

- Every compound in the input library yields a descriptor row — no missing SMILES/InChIKey parses
- Descriptor matrix row count matches the compound library size (1,021 pesticides for the training set)
- Train/test split is 80:20 and stratified on the RT distribution
- XGBoost test metrics reproduce the reported values: RMSE 0.89 min, R2 = 0.79, MAE = 0.60 min
- The fraction of XGBoost errors within 1 min of measured RT reaches the reported 95.1%, and all metrics are written to rt_prediction_metrics.csv

## Limitations

- Descriptor-based RT prediction is an in silico validation channel only; the article still treats analytical/stable-isotopically-labeled standard verification as the stronger validation, and in silico tools 'often fall short of providing direct evidence under real-world, omics-scale conditions'
- Scarcity of authentic standards for CMDN-annotated metabolites means predicted RT agreement is a surrogate, not direct, confirmation of chromatographic behavior
- Hyperparameters must be kept fixed/default for reproducibility, which limits optimization headroom

## Evidence

- [methods] Compute molecular descriptors from the SMILES/InChIKey strings (e.g. RDKit descriptors) to build the feature matrix for RT regression.: [evidence span withheld — anchor status: not_found]
