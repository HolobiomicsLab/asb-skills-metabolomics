---
name: fully-connected-module-design
description: Use when extending a graph neural network (e.g., SigmaCCS-style ECC layers)
  with tabular molecular features — fingerprints, descriptors, molecule types, instrument
  type — to improve generalizability of CCS predictions.
when_to_use_negative:
- Input is already a flat feature table with no graph representation needed — a plain
  MLP suffices without the ECC/GNN branch.
- The goal is strict reproduction of the original SigmaCCS architecture, which must
  remain identical and unmodified.
- Working with the GraphCCS GCN pipeline (PyTorch/DGL adduct graphs), which uses a
  different featurization and framework.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0081
tools:
- name: Mol2CCS
  role: 'The extended SigmaCCS architecture being implemented: parallel four-layer
    FC module concatenated with the GNN ECC outputs'
  repo: https://github.com/enveda/ccs-prediction
- name: SigmaCCS
  role: Base architecture whose three ECC layers and graph construction are kept identical
    in the extension
  repo: https://github.com/zmzhang/SigmaCCS
- name: RDKit (v2023.09.5)
  role: Reads SMILES representations to construct the molecular graphs consumed by
    the ECC layers
- name: Python
  role: Implementation and verification environment for the module (scripts/train-test.py
    with --dropout-rate 0.1)
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:cca44c6123fb86ff60d1e9be95c7b33d2993eec3f0d74be8ee4b0f535df1aa5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - SigmaCCS
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fully-connected-module-design

## Summary

Design a parallel fully connected module that ingests non-graph molecular features (fingerprints, descriptors, molecule type, instrument type) and concatenate it with a GNN branch, as done in Mol2CCS to extend SigmaCCS for CCS prediction.

## When to use

Use when extending a graph neural network (e.g., SigmaCCS-style ECC layers) with tabular molecular features — fingerprints, descriptors, molecule types, instrument type — to improve generalizability of CCS predictions.

## When NOT to use

- Input is already a flat feature table with no graph representation needed — a plain MLP suffices without the ECC/GNN branch.
- The goal is strict reproduction of the original SigmaCCS architecture, which must remain identical and unmodified.
- Working with the GraphCCS GCN pipeline (PyTorch/DGL adduct graphs), which uses a different featurization and framework.

## Inputs

- Molecular SMILES strings parsed into graphs via RDKit (v2023.09.5) for the ECC layers
- Seven additional per-molecule features: molecular fingerprints, molecular descriptors, molecule types, and instrument type
- Sample molecular graphs with CCS labels from METLIN-CCS or CCSBase (v1.3) for forward-pass validation

## Outputs

- mol2ccs_model.py containing the extended SigmaCCS architecture (ECC layers + parallel FC module + concatenation + 8x384 FC head)
- Forward-pass verification results confirming the module structure executes on sample graphs

## How to apply

Keep the GNN module (SigmaCCS's three ECC layers operating on molecular graphs built from SMILES with RDKit v2023.09.5) unchanged, and add a parallel branch of four fully connected layers with 256, 512, 512, and 256 neurons that processes the seven additional molecular features. Apply dropout (0.1, matching the training script's --dropout-rate) within this branch to prevent overfitting, which is critical given the limited experimental CCS data. Concatenate the ECC layer outputs with the parallel module outputs, then follow the joint representation with eight fully connected layers of 384 neurons each that regress the CCS value. Validate by running a forward pass on sample molecular graphs from METLIN-CCS or CCSBase to confirm the concatenated architecture executes correctly, and save the implementation (e.g., mol2ccs_model.py).

## Related tools

- **Mol2CCS** (The extended SigmaCCS architecture being implemented: parallel four-layer FC module concatenated with the GNN ECC outputs) — https://github.com/enveda/ccs-prediction
- **SigmaCCS** (Base architecture whose three ECC layers and graph construction are kept identical in the extension) — https://github.com/zmzhang/SigmaCCS
- **RDKit (v2023.09.5)** (Reads SMILES representations to construct the molecular graphs consumed by the ECC layers)
- **Python** (Implementation and verification environment for the module (scripts/train-test.py with --dropout-rate 0.1)) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- Forward pass on sample METLIN-CCS or CCSBase molecular graphs executes without shape errors through concatenation and the 8x384 FC head.
- The GNN/ECC module matches the original SigmaCCS implementation from zmzhang/SigmaCCS (unchanged graph branch).
- Parallel module has exactly four FC layers with 256/512/512/256 neurons and dropout 0.1 applied.
- Additional-feature branch consumes the seven non-graph features (fingerprints, descriptors, molecule types, instrument type).
- Trained model reproduces improved generalizability in cross-dataset evaluation (e.g., train-metlin-test-ccsbase runs via Makefile targets).

## Limitations

- Generalizability gains from added features are only partial; performance still declines on structurally novel regions.
- METLIN-CCS focuses on synthetic small molecules, so evaluation scope is restricted to specific regions of chemical space.
- Variations in experimental conditions (e.g., instrument types) can introduce inaccuracies in the CCS values used to train the model.

## Evidence

- [other] Mol2CCS extends SigmaCCS with a parallel module consisting of four fully connected layers (256, 512, 512, and 256 neurons) that applies dropout to prevent overfitting, while the GNN module remains identical to SigmaCCS.: [evidence span withheld — anchor status: not_found]
- [methods] Concatenate the ECC layer outputs with the parallel module outputs.: [evidence span withheld — anchor status: not_found]
- [methods] Read the SMILES representation of each molecule using RDKit (v2023.09.5) to construct the molecular graphs: [evidence span withheld — anchor status: not_found]
- [abstract] Generalization can be partially improved by extending models to account for additional features: "demonstrates how generalization can be partially improved by extending models to account for additional features such as molecular fingerprints, descriptors, and the molecule types"
- [readme] --dropout-rate 0.1: "--dropout-rate 0.1 \
	--epochs 400"
