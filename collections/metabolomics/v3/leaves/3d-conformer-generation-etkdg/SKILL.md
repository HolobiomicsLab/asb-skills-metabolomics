---
name: 3d-conformer-generation-etkdg
description: 'Use when a graph-based molecular model (e.g., SigmaCCS) needs 3D structural
  input: SMILES/adduct lists must be embedded to 3D conformers before convertToGraph
  and CCS prediction.'
when_to_use_negative:
- The downstream model consumes string-based representations directly (e.g., DeepCCS
  one-hot SMILES encoding, length <= 250 chars) and needs no 3D coordinates
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se, which
  SigmaCCS's atom set does not support
- Only 2D descriptors are required and no graph or 3D-geometry-dependent prediction
  follows
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3332
tools:
- name: SigmaCCS
  role: 'Consumes the generated conformers: GraphData.convertToGraph turns ETKDG/MMFF
    coordinates into node, adjacency, and edge feature matrices for Model_prediction'
  repo: https://github.com/zmzhang/SigmaCCS
- name: RDKit
  role: Performs ETKDGv3 embedding (EmbedMultipleConfs, numConfs=1, useRandomCoords=True,
    maxAttempts=1) and MMFF optimization (MMFFOptimizeMoleculeConfs, numThreads=0)
  repo: https://rdkit.org/
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_003
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:b498c317cba132c8361db5799311e8f6ce9511b53bf98112395ba2d1fb824e8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - CCSbase
  - DeepCCS
  - ISiCLE
  - RDKit
  - SigmaCCS
  - Spektral
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# 3D conformer generation with ETKDG

## Summary

Generate and minimize a single 3D conformer per molecule with RDKit ETKDGv3 followed by MMFF94 optimization, producing the 3D coordinates that SigmaCCS converts into molecular graphs (node feature, adjacency, and edge feature matrices) for graph-neural-network CCS prediction.

## When to use

Use when a graph-based molecular model (e.g., SigmaCCS) needs 3D structural input: SMILES/adduct lists must be embedded to 3D conformers before convertToGraph and CCS prediction.

## When NOT to use

- The downstream model consumes string-based representations directly (e.g., DeepCCS one-hot SMILES encoding, length <= 250 chars) and needs no 3D coordinates
- Molecules contain elements outside C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se, which SigmaCCS's atom set does not support
- Only 2D descriptors are required and no graph or 3D-geometry-dependent prediction follows

## Inputs

- SMILES strings (e.g., from TestData.csv, Supplementary Data 5/6, or PubChem compound lists)
- RDKit Mol objects
- Saved preprocessing parameters (parameter.pkl: adduct set, atoms set, coordinate min/max)

## Outputs

- One ETKDGv3-embedded, MMFF-optimized 3D conformer per molecule (conformer coordinate blocks)
- Molecular graph matrices via convertToGraph: node feature matrix, adjacency matrix, edge feature matrix
- Failure log of molecules where ETKDG embedding or MMFF94 minimization failed

## How to apply

For each SMILES string, parse the molecule with RDKit and embed one 3D conformer using AllChem.EmbedMultipleConfs with numConfs=1, useRandomCoords=True, and maxAttempts=1 under ETKDGv3 parameters. Immediately minimize the embedded conformer with AllChem.MMFFOptimizeMoleculeConfs (numThreads=0) so the coordinates are energy-relaxed, since CCS depends strongly on 3D geometry. Record which molecules fail embedding or MMFF minimization — the paper reports 33 of 94,161,896 PubChem molecules (and 5 of 5,645 CCSbase entries) failed ETKDG embedding and 662 conformers could not be MMFF94-minimized. Pass the resulting conformer coordinates, together with the adduct set and atom set loaded from parameter.pkl, into GraphData.convertToGraph to build the node feature matrix, adjacency matrix, and edge feature matrix consumed by the trained model.h5.

## Related tools

- **SigmaCCS** (Consumes the generated conformers: GraphData.convertToGraph turns ETKDG/MMFF coordinates into node, adjacency, and edge feature matrices for Model_prediction) — https://github.com/zmzhang/SigmaCCS
- **RDKit** (Performs ETKDGv3 embedding (EmbedMultipleConfs, numConfs=1, useRandomCoords=True, maxAttempts=1) and MMFF optimization (MMFFOptimizeMoleculeConfs, numThreads=0)) — https://rdkit.org/

## Examples

```
ps = AllChem.ETKDGv3(); ps.useRandomCoords=True; re = AllChem.EmbedMultipleConfs(mol, numConfs=1, params=ps); AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads=0); adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)
```

## Evaluation signals

- Each molecule yields exactly one conformer (numConfs=1) and the MMFF minimization step returns successfully before graph conversion
- The three matrices from convertToGraph (node features, adjacency, edge features) have consistent dimensions matching the molecule's atom and bond counts
- Downstream SigmaCCS predictions on test data reproduce reported metrics: R2 = 0.9945 and median relative error = 1.1751% on TestData.csv (559 molecules)
- Failed-embedding molecules are logged rather than silently dropped (expected failure rate ~33/94M for PubChem-scale, 5/5,645 for CCSbase entries)

## Limitations

- ETKG could not generate conformers for 33 of 94,161,896 PubChem molecules and 5 of 5,645 CCSbase entries, and 662 ETKDG-generated conformers could not be minimized by MMFF94
- SigmaCCS cannot predict molecules containing elements other than C, H, O, N, P, S, F, Cl, Br, I, Co, As, and Se
- Single-conformer (numConfs=1) embedding is a speed/accuracy trade-off; conformer quality directly bounds CCS prediction accuracy since CCS is closely tied to 3D structure

## Evidence

- [methods] ETKDG embedding parameters: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
- [methods] MMFF optimization call: "re = AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0)"
- [methods] Graph conversion consumes conformer coordinates: "adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms)"
- [discussion] Known failure rates for conformer generation: "the conformers of some molecules (33 out 94,161,896) in PubChem and (5 out 5645 entries) in CCSbase cannot be generated by the ETKDG method"
- [intro] 3D structure rationale for CCS prediction: "the CCS value of a molecule is closely related to its 3D structure, which is not included in the one-hot encoding of its SMILES string"
