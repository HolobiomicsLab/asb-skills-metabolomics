---
name: attribute-importance-quantification
description: Use when you must rank which node attributes of a trained graph neural
  network (e.g., SigmaCCS atom features in model.h5) drive CCS prediction accuracy,
  by masking each attribute and measuring the prediction-error increase on a test
  set.
when_to_use_negative:
- The model input is not a graph with distinct node attributes (e.g., DeepCCS one-hot
  encoded SMILES strings fed to a CNN), so masking atom attributes is not applicable.
- You need per-molecule or per-prediction attribution scores rather than a global
  ranking of node attributes; masking measures aggregate error increase across the
  whole test set.
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
  — the underlying SigmaCCS model cannot predict them, so masked errors would be meaningless.
edam_topics: []
tools:
- name: SigmaCCS
  role: Supplies the trained model.h5, parameter.pkl, TestData.csv, and GraphData.py
    convertToGraph used to construct the graph dataset and run baseline plus attribute-masked
    CCS predictions in the Attribute Analysis.ipynb procedure
  repo: https://github.com/zmzhang/SigmaCCS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:1e9dd4865567cd611fab6f767f43c0414ee87e4a7d42d9931cdc1719cf968e95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - SigmaCCS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# attribute-importance-quantification

## Summary

Model-agnostic feature-importance quantification for the SigmaCCS graph neural network: each of the five node attributes (atomic symbol, atomic degree, 3D coordinates, atomic radius, atomic mass) is masked one at a time and the resulting increase in CCS prediction error is normalized into percentage importances.

## When to use

Use when you must rank which node attributes of a trained graph neural network (e.g., SigmaCCS atom features in model.h5) drive CCS prediction accuracy, by masking each attribute and measuring the prediction-error increase on a test set.

## When NOT to use

- The model input is not a graph with distinct node attributes (e.g., DeepCCS one-hot encoded SMILES strings fed to a CNN), so masking atom attributes is not applicable.
- You need per-molecule or per-prediction attribution scores rather than a global ranking of node attributes; masking measures aggregate error increase across the whole test set.
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se — the underlying SigmaCCS model cannot predict them, so masked errors would be meaningless.

## Inputs

- Trained SigmaCCS model file model.h5
- parameter.pkl holding the adduct set, atoms set, and coordinate min/max used at graph construction
- TestData.csv with SMILES, 3D coordinates, adduct type, and experimental CCS values
- Graph dataset built by GraphData.py: node feature matrix, adjacency matrix, and edge feature matrix per molecule

## Outputs

- attribute_importance.csv with ranked percentage importances per atom attribute
- Expected ranking (Fig. 4a): atomic symbol 27.57%, atomic degree 26.86%, 3D coordinates 14.82%, atomic radius 9.56%, atomic mass 0.61%

## How to apply

Load the trained SigmaCCS model from model.h5 together with parameter.pkl (adduct set, atoms set, coordinate min/max). Read TestData.csv and build the graph dataset with GraphData.py, where convertToGraph(smiles, Coordinate, All_Atoms) yields the node feature matrix, adjacency matrix, and edge feature matrix per molecule. Run baseline CCS predictions with the unmodified model to obtain reference errors. For each node attribute in turn — atomic symbol, atomic degree, 3D coordinates, atomic radius, atomic mass — mask that attribute across all graphs, re-run predictions with the same model, and quantify the increase in prediction error caused by masking. Normalize the per-attribute error increases into percentage importances, mirroring the Attribute Analysis.ipynb procedure, and write the ranked percentages to attribute_importance.csv.

## Related tools

- **SigmaCCS** (Supplies the trained model.h5, parameter.pkl, TestData.csv, and GraphData.py convertToGraph used to construct the graph dataset and run baseline plus attribute-masked CCS predictions in the Attribute Analysis.ipynb procedure) — https://github.com/zmzhang/SigmaCCS

## Examples

```
adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)
```

## Evaluation signals

- Normalized per-attribute importances sum to approximately 100%.
- Ranking order matches atomic symbol > atomic degree > 3D coordinates > atomic radius > atomic mass.
- Percentages reproduce the reported Fig. 4a values within reasonable tolerance: 27.57% (atomic symbol), 26.86% (atomic degree), 14.82% (3D coordinates), 9.56% (atomic radius), 0.61% (atomic mass).
- Baseline CCS predictions from the unmodified model.h5 on TestData.csv match the reference test-set errors before any masking is applied.
- attribute_importance.csv contains one ranked row per masked attribute with a percentage importance column.

## Limitations

- Importances are specific to the trained SigmaCCS model and its test set (559 molecules); they are not transferable to other CCS predictors without re-running the masking analysis.
- Masking yields a global attribute ranking, not per-molecule or per-prediction explanations of individual CCS errors.
- The analysis inherits SigmaCCS coverage limits: only three adduct ion types ([M+H]+, [M+Na]+, [M-H]-) and elements C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se are supported.

## Evidence

- [abstract] Model-agnostic interpretation of SigmaCCS: "The model-agnostic interpretation method and the visualization of the learned representations were used to investigate the chemical rationality of SigmaCCS"
- [methods] convertToGraph builds the three graph matrices: "adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)"
- [readme] SMILES-to-Graph conversion via GraphData.py: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph. The related method is shown in `GraphData.py`"
- [discussion] Element coverage limitation of SigmaCCS: "Although SigmaCCS cannot predict molecules of elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se"
