---
name: mmff-conformer-minimization
description: 'Use when preparing 3D molecular structures for CCS or other geometry-dependent
  ML prediction: after ETKDG conformer generation and before converting SMILES-derived
  molecules into graph matrices.'
when_to_use_negative:
- Molecules whose ETKDG embedding failed (33 of 94,161,896 PubChem entries in the
  paper) — no conformer exists to minimize
- Molecules containing elements outside the model's supported set (C, H, O, N, P,
  S, F, Cl, Br, I, Co, As, Se) — geometry refinement would not rescue them for SigmaCCS
- Workflows using descriptor- or SMILES-encoding-based predictors (e.g. DeepCCS one-hot
  CNNs, AllCCS descriptors) that do not consume 3D coordinates
edam_operation: http://edamontology.org/operation_0480
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_3047
tools:
- name: RDKit
  role: Provides AllChem.MMFFOptimizeMoleculeConfs for force-field minimization of
    ETKDG-generated conformers and AllChem.EmbedMultipleConfs for the preceding embedding
    step
  repo: https://github.com/zmzhang/SigmaCCS
- name: SigmaCCS
  role: 'Downstream consumer: minimized conformers are converted via convertToGraph
    (GraphData.py) into graph inputs for CCS prediction of [M+H]+, [M-H]-, and [M+Na]+
    adducts'
  repo: https://github.com/zmzhang/SigmaCCS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42004-023-00939-w
    title: Highly accurate and large-scale collision cross sections prediction with
      graph neural networks
  source_content_hash: sha256:8acb314545f13721d79047020ae52aff0e3ea32f4fc7b50552c96101b3fbebc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - RDKit
  - SigmaCCS
  - Spektral
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0480
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_3047
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# MMFF conformer minimization

## Summary

Minimizes RDKit-generated 3D conformers with the MMFF94 force field so that SigmaCCS graph inputs (node, adjacency, and edge feature matrices) are built on physically relaxed coordinates rather than raw ETKDG embeddings.

## When to use

Use when preparing 3D molecular structures for CCS or other geometry-dependent ML prediction: after ETKDG conformer generation and before converting SMILES-derived molecules into graph matrices.

## When NOT to use

- Molecules whose ETKDG embedding failed (33 of 94,161,896 PubChem entries in the paper) — no conformer exists to minimize
- Molecules containing elements outside the model's supported set (C, H, O, N, P, S, F, Cl, Br, I, Co, As, Se) — geometry refinement would not rescue them for SigmaCCS
- Workflows using descriptor- or SMILES-encoding-based predictors (e.g. DeepCCS one-hot CNNs, AllCCS descriptors) that do not consume 3D coordinates

## Inputs

- RDKit mol objects with a single ETKDG-generated 3D conformer
- SMILES strings loaded from a PubChem compound list (MW < 1500 Da, supported elements only)

## Outputs

- MMFF94-minimized 3D conformers ready for graph conversion
- Graph matrices (node features, adjacency, edge features) and normalized coordinates for SigmaCCS prediction

## How to apply

Generate one 3D conformer per molecule with RDKit ETKDGv3 via AllChem.EmbedMultipleConfs (numConfs=1, maxAttempts=1, useRandomCoords=True), then relax it with AllChem.MMFFOptimizeMoleculeConfs (numThreads=0) as in the SigmaCCS data pre-processing. The minimization matters because CCS values are closely tied to 3D structure, so raw ETKDG coordinates would inject geometric noise into the node features and coordinate min/max normalization parameters. Skip molecules whose conformers MMFF94 cannot minimize (662 molecules in the paper failed this step) and record them as coverage gaps. Only then convert the optimized conformer to graph matrices with convertToGraph in GraphData.py, using the saved parameter.pkl (adduct set, atoms set, coordinate min/max).

## Related tools

- **RDKit** (Provides AllChem.MMFFOptimizeMoleculeConfs for force-field minimization of ETKDG-generated conformers and AllChem.EmbedMultipleConfs for the preceding embedding step) — https://github.com/zmzhang/SigmaCCS
- **SigmaCCS** (Downstream consumer: minimized conformers are converted via convertToGraph (GraphData.py) into graph inputs for CCS prediction of [M+H]+, [M-H]-, and [M+Na]+ adducts) — https://github.com/zmzhang/SigmaCCS

## Examples

```
from rdkit import Chem; from rdkit.Chem import AllChem; mol = Chem.AddHs(Chem.MolFromSmiles(smiles)); AllChem.EmbedMultipleConfs(mol, numConfs=1); AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads=0)
```

## Evaluation signals

- MMFFOptimizeMoleculeConfs returns a (converged, energy) tuple per molecule; verify convergence flags and that returned energies are finite and non-increasing
- Minimized coordinates should yield finite values for the coordinate min/max parameters saved in parameter.pkl used to normalize graph inputs
- Count molecules skipped due to MMFF minimization failure and confirm the rate is small (paper: 662 of ~94M PubChem molecules could not be minimized)
- Downstream check: predicted CCS values on compounds with known experimental CCS fall within the reported model accuracy (median relative error ~1.1751%, R2 ~0.9945)

## Limitations

- ETKG-generated conformers of 662 molecules could not be minimized by MMFF94, so those molecules are dropped from the database build
- ETKG itself fails on a small fraction of structures (33/94,161,896 PubChem molecules; 5/5,645 CCSbase entries), leaving nothing to minimize
- Minimization is one conformer per molecule (numConfs=1, maxAttempts=1, useRandomCoords=True), so conformational sampling breadth is traded for throughput

## Evidence

- [methods] MMFF minimization step in SigmaCCS pre-processing: "re = AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0)"
- [methods] Preceding ETKDG embedding step: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)"
- [intro] Rationale for accurate 3D geometry: "the CCS value of a molecule is closely related to its 3D structure, which is not included in the one-hot encoding of its SMILES string."
- [discussion] MMFF94 minimization failure rate: "There are 662 molecules whose conformers generated by ETKDG cannot be minimized by the MMFF94 method."
- [readme] Downstream graph conversion consuming minimized conformers: "SigmaCCS is a model for predicting CCS based on graph neural networks, so we need to convert SMILES strings to Graph. The related method is shown in `GraphData.py`"
