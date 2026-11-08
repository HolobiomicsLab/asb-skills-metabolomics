---
name: ccs-prediction-model-evaluation
description: Use when you need to quantify how well a CCS prediction model (e.g. SigmaCCS
  vs CCSbase) performs on an external test set or a specialized dataset such as plant
  metabolites, using R2, RMSE, and Median RE metrics.
when_to_use_negative:
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se —
  SigmaCCS cannot predict them
- Adduct types beyond [M+H]+, [M-H]-, [M+Na]+ are needed — SigmaCCS encodes only these
  three adduct ion types
- Input is already aggregated performance data; no raw SMILES/CCS pairs exist to regenerate
  conformers and predictions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0154
tools:
- name: SigmaCCS
  role: Graph-neural-network predictor evaluated via Model_prediction with model.h5
    and parameter.pkl on ETKDGv3-derived molecular graphs
  repo: https://github.com/zmzhang/SigmaCCS
- name: CCSbase
  role: Comparison baseline that assigns structures to K-Means clusters and predicts
    CCS with cluster-specific regression models
- name: RDKit
  role: Generates and optimizes 3D conformers (ETKDGv3 EmbedMultipleConfs, MMFFOptimizeMoleculeConfs)
    used to build molecular graphs
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:a7daac6449aac1801929fe852cfbd9b34c48822eb56a6f7ad31faddac73f1cb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - CCSbase
  - RDKit
  - SigmaCCS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ccs-prediction-model-evaluation

## Summary

Benchmark CCS predictors such as SigmaCCS and CCSbase on held-out datasets by computing R2, RMSE, and median relative error against experimental values, and optionally probe model rationality via attribute-masking feature importance.

## When to use

Use when you need to quantify how well a CCS prediction model (e.g. SigmaCCS vs CCSbase) performs on an external test set or a specialized dataset such as plant metabolites, using R2, RMSE, and Median RE metrics.

## When NOT to use

- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se — SigmaCCS cannot predict them
- Adduct types beyond [M+H]+, [M-H]-, [M+Na]+ are needed — SigmaCCS encodes only these three adduct ion types
- Input is already aggregated performance data; no raw SMILES/CCS pairs exist to regenerate conformers and predictions

## Inputs

- Benchmark table (CSV, e.g. Supplementary Data 5/6) with SMILES, adduct type ([M+H]+, [M-H]-, [M+Na]+), and experimental CCS values
- Trained SigmaCCS model file (model.h5)
- SigmaCCS parameter pickle (parameter.pkl: adduct set, atoms set, coordinate min/max)
- TestData.csv for attribute-masking feature importance
- CCSbase predictions or its K-Means clustering + cluster-specific models

## Outputs

- external_test_evaluation.csv with R2, RMSE, Median RE for SigmaCCS and CCSbase on the 294-entry external test set
- ccs_evaluation_metrics.csv with R2, RMSE, Median RE on the plant dataset
- attribute_importance.csv with ranked masking-based feature importances
- Predicted-vs-experimental CCS scatter plots (Supplementary Fig. 3a-d)

## How to apply

Load a benchmark table of SMILES, adduct type ([M+H]+, [M-H]-, or [M+Na]+), and experimental CCS values (e.g. the 294-entry deduplicated external test set or the 114-entry plant dataset). For each molecule, generate a 3D conformer with RDKit ETKDGv3 (EmbedMultipleConfs with numConfs=1, useRandomCoords=True, maxAttempts=1), optimize with MMFFOptimizeMoleculeConfs, and build the molecular graph (node features, adjacency, edge features) via GraphData.convertToGraph. Run each predictor — SigmaCCS via Model_prediction with the trained model.h5 and parameter.pkl, and CCSbase via K-Means cluster assignment followed by cluster-specific regression — then compute R2, RMSE, and median relative error (%) with scikit-learn regression metrics. To probe chemical rationality, mask one node attribute at a time (atomic symbol, atomic degree, 3D coordinates, atomic radius, atomic mass), re-predict, and normalize the error increases into percentage importances. Assemble metrics into a CSV (e.g. external_test_evaluation.csv, ccs_evaluation_metrics.csv, attribute_importance.csv) and optionally plot predicted-vs-experimental scatter plots.

## Related tools

- **SigmaCCS** (Graph-neural-network predictor evaluated via Model_prediction with model.h5 and parameter.pkl on ETKDGv3-derived molecular graphs) — https://github.com/zmzhang/SigmaCCS
- **CCSbase** (Comparison baseline that assigns structures to K-Means clusters and predicts CCS with cluster-specific regression models)
- **RDKit** (Generates and optimizes 3D conformers (ETKDGv3 EmbedMultipleConfs, MMFFOptimizeMoleculeConfs) used to build molecular graphs)

## Examples

```
re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps); re = AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0); adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)
```

## Evaluation signals

- External test set metrics match reported values within numerical tolerance: SigmaCCS R2=0.9780, RMSE=6.7012, Median RE=1.8211%; CCSbase R2=0.9778, RMSE=6.7240, Median RE=1.3608%
- Plant dataset metrics match: SigmaCCS R2=0.9655, RMSE=5.3812, Median RE=1.4232%; CCSbase R2=0.9643, RMSE=5.4720, Median RE=2.3211%
- Attribute importances reproduce reported ranking within tolerance: atomic symbol 27.57%, atomic degree 26.86%, 3D coordinates 14.82%, atomic radius 9.56%, atomic mass 0.61%
- Predictor columns are complete (no dropped entries) for datasets where conformers were successfully generated; ETKDG failures should be reported explicitly
- Output CSVs contain exactly the columns Metric, SigmaCCS, CCSbase (or per-attribute FI percentages) with values in plausible ranges (R2 in [0,1], Median RE in %)

## Limitations

- ETKDG conformer generation failed for 33/94,161,896 PubChem molecules and 5/5,645 CCSbase entries, and MMFF94 minimization failed for 662 molecules
- CCSbase can suffer large deviations when a molecule is assigned to an unsuitable K-Means cluster
- SigmaCCS supports only three adduct ion types; encoding more adducts is noted as future work

## Evidence

- [results] SigmaCCS vs CCSbase metrics on deduplicated external test set: [evidence span withheld — anchor status: not_found]
- [results] plant dataset comparison: [evidence span withheld — anchor status: not_found]
- [results] feature importance masking: [evidence span withheld — anchor status: not_found]
- [discussion] CCSbase method: "CCSbase uses K-Means clustering for the untargeted classification of chemical structures and then performs CCS predictions using specific models trained on the corresponding cluster data."
- [readme] conformer pipeline in repo: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph."
- [discussion] ETKDG failures: "the conformers of some molecules (33 out 94,161,896) in PubChem and (5 out 5645 entries) in CCSbase cannot be generated by the ETKDG method"
