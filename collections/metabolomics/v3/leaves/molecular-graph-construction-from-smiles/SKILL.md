---
name: molecular-graph-construction-from-smiles
description: Use when raw input is SMILES strings with adduct annotations ([M+H]+,
  [M-H]-, [M+Na]+) and CCS values must be predicted by a GNN that requires 3D-informed
  molecular graphs rather than descriptors or one-hot SMILES.
when_to_use_negative:
- The molecule contains elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and
  Se — SigmaCCS cannot predict these
- The downstream predictor consumes descriptors or one-hot SMILES (e.g., DeepCCS/AllCCS),
  so graph construction adds no benefit
- Conformers cannot be embedded by ETKDG or minimized by MMFF94 (failed for 33/94,161,896
  PubChem molecules and 662 molecules in the paper)
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3474
tools:
- name: SigmaCCS
  role: Provides GraphData.convertToGraph and parameter.pkl used to build node/adjacency/edge
    feature matrices from ETKDGv3/MMFF94 conformers
  repo: https://github.com/zmzhang/SigmaCCS
- name: RDKit
  role: Generates 3D conformers via EmbedMultipleConfs (ETKDGv3) and minimizes them
    with MMFFOptimizeMoleculeConfs before graph conversion
  repo: https://rdkit.org/
- name: Spektral
  role: Consumes the constructed graph matrices (with TensorFlow model.h5) for GNN-based
    CCS prediction
  repo: https://graphneural.network/
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_008
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:23f2154073f271221db4e5cf4b1bb23fd3edf9d620288e72e2d2b3ed2f2a583f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - CCSbase
  - ISiCLE
  - RDKit
  - SigmaCCS
  - Spektral
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Molecular Graph Construction from SMILES

## Summary

Converts SMILES strings into molecular graphs (node feature matrix, adjacency matrix, edge feature matrix) for graph-neural-network CCS prediction in SigmaCCS, by first embedding a 3D conformer with RDKit ETKDGv3 and minimizing it with MMFF94.

## When to use

Use when raw input is SMILES strings with adduct annotations ([M+H]+, [M-H]-, [M+Na]+) and CCS values must be predicted by a GNN that requires 3D-informed molecular graphs rather than descriptors or one-hot SMILES.

## When NOT to use

- The molecule contains elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se — SigmaCCS cannot predict these
- The downstream predictor consumes descriptors or one-hot SMILES (e.g., DeepCCS/AllCCS), so graph construction adds no benefit
- Conformers cannot be embedded by ETKDG or minimized by MMFF94 (failed for 33/94,161,896 PubChem molecules and 662 molecules in the paper)

## Inputs

- SMILES strings with adduct type annotations ([M+H]+, [M-H]-, or [M+Na]+)
- CSV files such as TrainData.csv or TestData.csv with SMILES, adduct, and experimental CCS values
- parameter.pkl (adduct set, atoms set, coordinate min/max) from the SigmaCCS repository

## Outputs

- Node feature matrix per molecule (atomic symbol, degree, 3D coordinates, atomic radius, atomic mass)
- Adjacency matrix per molecule
- Edge feature matrix per molecule
- MyDataset graph collection aligned with adduct one-hot encodings

## How to apply

For each molecule, generate one 3D conformer with RDKit ETKDGv3 (AllChem.EmbedMultipleConfs with numConfs=1, useRandomCoords=True, maxAttempts=1, randomSeed=-1) and minimize it with AllChem.MMFFOptimizeMoleculeConfs (numThreads=0). Load the saved parameter.pkl containing the adduct set, atoms set, and coordinate min/max normalization values, then call GraphData.convertToGraph(smiles, Coordinate, All_Atoms) to produce the three matrices: node features, adjacency, and edge features. Assemble entries into a MyDataset so graphs pair with adduct one-hot vectors before feeding the trained model.h5. The rationale is that CCS depends closely on 3D structure, which SMILES one-hot encoding alone does not capture, so node features include atomic symbol, degree, mass, radius, and 3D coordinates.

## Related tools

- **SigmaCCS** (Provides GraphData.convertToGraph and parameter.pkl used to build node/adjacency/edge feature matrices from ETKDGv3/MMFF94 conformers) — https://github.com/zmzhang/SigmaCCS
- **RDKit** (Generates 3D conformers via EmbedMultipleConfs (ETKDGv3) and minimizes them with MMFFOptimizeMoleculeConfs before graph conversion) — https://rdkit.org/
- **Spektral** (Consumes the constructed graph matrices (with TensorFlow model.h5) for GNN-based CCS prediction) — https://graphneural.network/

## Examples

```
from GraphData import convertToGraph; re = AllChem.EmbedMultipleConfs(mol, numConfs=1, params=ps); AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads=0); adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)
```

## Evaluation signals

- Each molecule yields three matrices (features, adjacency, edge_features) with node attributes covering atomic symbol, degree, 3D coordinates, atomic radius, and atomic mass
- Conformer generation settings match the paper: numConfs=1, maxAttempts=1, useRandomCoords=True, followed by MMFFOptimizeMoleculeConfs minimization
- Graphs built for TestData.csv feed a model achieving R2 = 0.9945 and median relative error of 1.1751% on the test set
- Coordinate values are normalized using the min/max saved in parameter.pkl; adduct one-hot vectors align with the three supported ion types
- Masking-based feature importance over the node attributes should rank atomic symbol (27.57%) and degree (26.86%) highest, confirming attributes are encoded as described

## Limitations

- ETKDG failed to generate conformers for 33 of 94,161,896 PubChem molecules and 5 of 5,645 CCSbase entries; 662 molecules' conformers could not be minimized by MMFF94
- Only molecules with elements C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se are supported by the atoms set in parameter.pkl
- The graph encodes only three adduct ion types; encoding more adducts is noted as future work

## Evidence

- [methods] ETKDG conformer settings: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
- [methods] MMFF minimization: "re = AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0)"
- [methods] Graph conversion call: "adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)"
- [readme] README rationale for graph conversion: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph."
- [intro] 3D structure motivation: "the CCS value of a molecule is closely related to its 3D structure, which is not included in the one-hot encoding of its SMILES string."
