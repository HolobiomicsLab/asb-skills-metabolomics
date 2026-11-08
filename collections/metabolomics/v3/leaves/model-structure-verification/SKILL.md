---
name: model-structure-verification
description: Use when re-implementing or adapting a published CCS prediction architecture
  (e.g., extending SigmaCCS with fingerprints, descriptors, molecule type, or instrument
  type features) and you must confirm the assembled modules execute on real molecular
  graphs before training.
when_to_use_negative:
- You only need to benchmark pre-existing published models (SigmaCCS, GraphCCS, Mol2CCS)
  rather than reconstruct or modify an architecture — train and evaluate them as-is
  instead.
- Input is already trained model weights or prediction outputs (e.g., Zenodo prediction
  archives) rather than SMILES/features to build a network from.
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: RDKit (v2023.09.5)
  role: Parses SMILES to build molecular graphs feeding the SigmaCCS-style ECC layers
- name: SigmaCCS
  role: Reference architecture whose three ECC layers are reimplemented unchanged
    in the GNN module
  repo: https://github.com/zmzhang/SigmaCCS
- name: Mol2CCS
  role: 'Target architecture: SigmaCCS plus the parallel 256/512/512/256 fully connected
    feature branch being verified'
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Implementation and forward-pass verification language for the model modules
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:a60ec75b3196782b012d87b8e8983e77ed9a95840d9f32fbfd2e130910bfbe42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - SigmaCCS
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Model structure verification

## Summary

Reconstructs and sanity-checks a GNN-based CCS prediction architecture (Mol2CCS as an extension of SigmaCCS) by wiring the three ECC graph layers of SigmaCCS in parallel with a four-layer fully connected module (256/512/512/256 neurons, dropout 0.1) for seven extra features, concatenating them, and confirming the assembled network executes a correct forward pass on sample molecular graphs.

## When to use

Use when re-implementing or adapting a published CCS prediction architecture (e.g., extending SigmaCCS with fingerprints, descriptors, molecule type, or instrument type features) and you must confirm the assembled modules execute on real molecular graphs before training.

## When NOT to use

- You only need to benchmark pre-existing published models (SigmaCCS, GraphCCS, Mol2CCS) rather than reconstruct or modify an architecture — train and evaluate them as-is instead.
- Input is already trained model weights or prediction outputs (e.g., Zenodo prediction archives) rather than SMILES/features to build a network from.

## Inputs

- SMILES strings of molecules from METLIN-CCS or CCSBase (v1.3) datasets
- Seven additional per-molecule features: molecular fingerprints, descriptors, molecule types, and instrument type
- Adduct type per molecule ([M+H]+, [M+Na]+, [M-H]- as in SigmaCCS)
- SigmaCCS reference implementation (zmzhang/SigmaCCS) defining the ECC layers

## Outputs

- Verified Mol2CCS module structure saved as mol2ccs_model.py
- Successful forward-pass confirmation on sample molecular graphs (no shape/runtime errors)
- Ready-to-train model object compatible with scripts/train-test.py and .h5 model output

## How to apply

Read SMILES with RDKit (v2023.09.5), generate 3D conformations via ETKDGv3 with MMFF optimization as in the SigmaCCS pipeline, and convert to graph form (node feature, adjacency, edge feature matrices) using the SigmaCCS GraphData utilities. Implement the three ECC layers exactly as in the zmzhang/SigmaCCS repository, keeping the GNN module identical to SigmaCCS. Add the parallel fully connected branch — four layers of 256, 512, 512, 256 neurons with dropout 0.1 — that processes the seven additional features (molecular fingerprints, descriptors, molecule types, instrument type), then concatenate ECC and parallel-branch outputs and feed them through eight fully connected layers of 384 neurons each. As the decision point, run a forward pass on sample molecular graphs drawn from METLIN-CCS or CCSBase: the pass must execute without shape errors before saving the implementation (e.g., mol2ccs_model.py) and moving on to training with the train-test.py script.

## Related tools

- **RDKit (v2023.09.5)** (Parses SMILES to build molecular graphs feeding the SigmaCCS-style ECC layers)
- **SigmaCCS** (Reference architecture whose three ECC layers are reimplemented unchanged in the GNN module) — https://github.com/zmzhang/SigmaCCS
- **Mol2CCS** (Target architecture: SigmaCCS plus the parallel 256/512/512/256 fully connected feature branch being verified) — https://github.com/enveda/ccs-prediction
- **Python** (Implementation and forward-pass verification language for the model modules)

## Examples

```
adj, features, edge_features = convertToGraph(smiles, Coordinate, All_Atoms); out = mol2ccs_model([features, adj, edge_features, extra_features])
```

## Evaluation signals

- A forward pass on sample METLIN-CCS or CCSBase molecular graphs completes without tensor shape errors through the concatenated ECC + fully connected representation.
- The parallel branch has exactly four fully connected layers of 256, 512, 512, 256 neurons with dropout 0.1; the post-concatenation stack has eight layers of 384 neurons each.
- The GNN module matches SigmaCCS's three ECC layers implemented as in the zmzhang/SigmaCCS repository.
- Graph construction from SMILES reproduces the three SigmaCCS matrices (node feature, adjacency, edge feature) using RDKit ETKDGv3 + MMFF-optimized 3D conformers.
- The saved mol2ccs_model.py is loadable and trainable via the repository's Makefile / scripts/train-test.py entry point.

## Limitations

- The GNN module must remain identical to SigmaCCS; generalization gains are only partial, and high accuracy on CCSBase does not generalize to structurally novel chemical spaces.
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values used to fit the model.
- Evaluation scope is limited mainly to small molecules and specific regions of chemical space (e.g., METLIN-CCS is based on synthetic structures).

## Evidence

- [intro] we first read the SMILES representation of each molecule using RDKit (v2023.09.5): "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
- [discussion] we benchmarked two GNNs ... and Mol2CCS: "we benchmarked two GNNs (i.e., SigmaCCS and GraphCCS) and Mol2CCS, an adaptation of SigmaCCS, on METLIN-CCS and CCSBase"
- [discussion] Mol2CCS extension includes additional features: "we demonstrate how an extension of the architecture of GNN (Mol2CCS) including other additional features improves the generalizability of SigmaCCS"
- [readme] SigmaCCS converts SMILES to graphs: [evidence span withheld — anchor status: external_doc]
- [readme] Generate 3D conformations with ETKDGv3 and MMFF optimization: "re = AllChem.EmbedMultipleConfs(mol, numConfs = 1, params = ps)
    re = AllChem.MMFFOptimizeMoleculeConfs(mol, numThreads = 0)"
