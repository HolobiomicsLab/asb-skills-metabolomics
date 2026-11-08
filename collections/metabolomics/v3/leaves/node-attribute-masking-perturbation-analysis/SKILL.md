---
name: node-attribute-masking-perturbation-analysis
description: Use when you need to determine which node attributes of a trained GNN
  (e.g. SigmaCCS CCS predictions) most affect prediction accuracy by perturbing each
  input feature and measuring the resulting performance drop.
when_to_use_negative:
- You only have descriptor-based (e.g. AllCCS) or SMILES-encoding (e.g. DeepCCS) models
  with no per-node attribute graph representation to mask.
- The goal is CCS prediction or filtering of candidate molecules (use the Prediction/Filtering
  workflow), not model interpretability.
- The model input is not a molecular graph with distinct node attributes — masking
  one node attribute is only meaningful when attributes are separable in the feature
  matrix.
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: SigmaCCS
  role: Trained GNN model whose node attributes (atomic symbol, atomic degree, 3D
    coordinates, atomic radius, atomic mass) are masked one at a time to quantify
    importance
  repo: https://github.com/zmzhang/SigmaCCS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:bd629323ce88656a2fb53cdf0e2957d1215061efd231fe9bb41fd231e35727f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - SigmaCCS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# node-attribute-masking-perturbation-analysis

## Summary

A model-agnostic interpretation method that quantifies node-attribute importance in a graph neural network (SigmaCCS) by masking one node attribute at a time in the graph dataset and measuring the increase in CCS prediction error relative to the unmasked model.

## When to use

Use when you need to determine which node attributes of a trained GNN (e.g. SigmaCCS CCS predictions) most affect prediction accuracy by perturbing each input feature and measuring the resulting performance drop.

## When NOT to use

- You only have descriptor-based (e.g. AllCCS) or SMILES-encoding (e.g. DeepCCS) models with no per-node attribute graph representation to mask.
- The goal is CCS prediction or filtering of candidate molecules (use the Prediction/Filtering workflow), not model interpretability.
- The model input is not a molecular graph with distinct node attributes — masking one node attribute is only meaningful when attributes are separable in the feature matrix.

## Inputs

- Trained SigmaCCS model file (model.h5)
- Saved parameter pickle (parameter.pkl: adduct set, atoms set, coordinate min/max)
- TestData.csv (SMILES, adduct, experimental CCS values)
- Graph dataset per molecule (node feature matrix, adjacency matrix, edge feature matrix) built by convertToGraph

## Outputs

- attribute_importance.csv with per-attribute importance percentages (atomic symbol 27.57%, atomic degree 26.86%, 3D coordinates 14.82%, atomic radius 9.56%, atomic mass 0.61%)
- Baseline vs. masked prediction error comparison per attribute

## How to apply

Load the trained model (model.h5) and saved parameters (parameter.pkl containing the adduct set, atoms set, and coordinate min/max), then read TestData.csv and build the graph dataset with GraphData.py's convertToGraph, which produces the node feature matrix, adjacency matrix, and edge feature matrix per molecule. First run baseline CCS predictions with the unmodified model to obtain reference errors. Then, for each node attribute in turn — atomic symbol, atomic degree, 3D coordinates, atomic radius, atomic mass — mask that attribute across the entire graph dataset, re-run CCS predictions with the same model, and quantify the increase in prediction error caused by masking. Normalize the per-attribute error increases into percentage importances, mirroring the Attribute Analysis.ipynb procedure, and write the ranked percentages to attribute_importance.csv.

## Related tools

- **SigmaCCS** (Trained GNN model whose node attributes (atomic symbol, atomic degree, 3D coordinates, atomic radius, atomic mass) are masked one at a time to quantify importance) — https://github.com/zmzhang/SigmaCCS

## Examples

```
adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)
```

## Evaluation signals

- Masked-at-a-time importances reproduce the reported ranking within reasonable tolerance: atomic symbol 27.57%, atomic degree 26.86%, 3D coordinates 14.82%, atomic radius 9.56%, atomic mass 0.61% (Fig. 4a).
- Baseline (unmasked) prediction errors on the test set match the original model's test-set performance (R² = 0.9945, median relative error 1.1751%).
- The five per-attribute importance percentages sum to approximately 100% after normalization.
- Each masked run uses the identical model weights (model.h5); only the node feature matrix of the graph dataset changes.
- Output attribute_importance.csv contains one ranked entry per masked attribute.

## Limitations

- Masking is a model-agnostic perturbation: importance reflects sensitivity of the trained model to each attribute, not a guaranteed causal chemical mechanism.
- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se, so the analysis is limited to molecules built from these atoms.
- Only three adduct ion types ([M+H]+, [M+Na]+, [M-H]-) are encoded; attributes relevant to other adducts are not captured.
- Conformer generation (ETKDG) or MMFF94 minimization may fail for some molecules (e.g. 33/94,161,896 PubChem and 5/5,645 CCSbase entries; 662 conformers could not be MMFF94-minimized), excluding them from the graph dataset.

## Evidence

- [abstract] model-agnostic interpretation: "The model-agnostic interpretation method and the visualization of the learned representations were used to investigate the chemical rationality of SigmaCCS"
- [methods] convertToGraph signature: "adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)"
- [methods] parameter saving: "Save relevant parameters. For details, see`parameter.py`."
- [readme] graph pre-processing in README: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph. The related method is shown in `GraphData.py`"
