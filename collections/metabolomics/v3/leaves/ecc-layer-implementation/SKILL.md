---
name: ecc-layer-implementation
description: Use when reconstructing SigmaCCS- or Mol2CCS-style CCS predictors from
  SMILES+adduct inputs, i.e. when the architecture must combine graph-based ECC layers
  with a parallel dense feature branch.
when_to_use_negative:
- Input is already a feature table without SMILES/molecular graphs — the ECC layers
  require graph construction from SMILES and adduct
- The task is benchmarking existing pretrained GNNs (SigmaCCS, GraphCCS) rather than
  implementing/reconstructing the architecture
- The prediction problem does not involve collision cross section (e.g. retention
  time or MS/MS spectra), since the ECC configuration here is tuned for CCS prediction
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3332
tools:
- name: RDKit (v2023.09.5)
  role: Reads SMILES and constructs the molecular graphs fed to the SigmaCCS-style
    ECC layers
  repo: https://github.com/zmzhang/SigmaCCS
- name: SigmaCCS
  role: Reference implementation of the three ECC layers that are reproduced identically
    in the GNN module
  repo: https://github.com/zmzhang/SigmaCCS
- name: Mol2CCS
  role: 'Extended architecture being implemented: parallel four-layer FC module (256/512/512/256,
    dropout 0.1) concatenated with ECC outputs'
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Runs the training/verification scripts that exercise the implemented module
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:1e16156d48c8e94839ce02fe3a1efe4ed67b619d83e03455e5729050ff7ea6cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - SigmaCCS
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ECC layer implementation (SigmaCCS/Mol2CCS graph module)

## Summary

Implements the edge-conditioned convolution (ECC) graph neural network layers of SigmaCCS, and their extension in Mol2CCS where ECC outputs are concatenated with a parallel four-layer fully connected module (256/512/512/256 neurons, dropout 0.1) that ingests molecular fingerprints, descriptors, molecule types and instrument type.

## When to use

Use when reconstructing SigmaCCS- or Mol2CCS-style CCS predictors from SMILES+adduct inputs, i.e. when the architecture must combine graph-based ECC layers with a parallel dense feature branch.

## When NOT to use

- Input is already a feature table without SMILES/molecular graphs — the ECC layers require graph construction from SMILES and adduct
- The task is benchmarking existing pretrained GNNs (SigmaCCS, GraphCCS) rather than implementing/reconstructing the architecture
- The prediction problem does not involve collision cross section (e.g. retention time or MS/MS spectra), since the ECC configuration here is tuned for CCS prediction

## Inputs

- SMILES strings of molecules (from METLIN-CCS or CCSBase datasets)
- Adduct type per molecule ([M+H]+, [M+Na]+, [M-H]-)
- Seven additional molecular features (molecular fingerprints, descriptors, molecule types, instrument type)
- 3D coordinates (optional; generated from SMILES via RDKit ETKDGv3/MMFF if absent)

## Outputs

- mol2ccs_model.py containing the SigmaCCS-equivalent ECC module plus the parallel dense branch and 8x384-neuron head
- Trained model file (e.g. model/train-metlin-test-metlin.h5) with parameter JSON
- Forward-pass verification results on sample molecular graphs

## How to apply

Read each molecule's SMILES with RDKit (v2023.09.5), generate 3D conformations (ETKDGv3 with numConfs=1, useRandomCoords=True, followed by MMFF optimization as in the SigmaCCS README), and convert to graph form via the node feature matrix, adjacency matrix and edge feature matrix (sigma/GraphData.py). Implement the three ECC layers exactly as in the original zmzhang/SigmaCCS repository, keeping the GNN module identical to SigmaCCS. For Mol2CCS, add a parallel branch of four fully connected layers (256, 512, 512, 256 neurons, dropout 0.1) processing the seven additional features (fingerprints, descriptors, molecule types, instrument type), concatenate the ECC and dense branch outputs, and follow with eight fully connected layers of 384 neurons each. Validate by running a forward pass on sample molecular graphs from METLIN-CCS or CCSBase to confirm the module executes correctly, then save as mol2ccs_model.py.

## Related tools

- **RDKit (v2023.09.5)** (Reads SMILES and constructs the molecular graphs fed to the SigmaCCS-style ECC layers) — https://github.com/zmzhang/SigmaCCS
- **SigmaCCS** (Reference implementation of the three ECC layers that are reproduced identically in the GNN module) — https://github.com/zmzhang/SigmaCCS
- **Mol2CCS** (Extended architecture being implemented: parallel four-layer FC module (256/512/512/256, dropout 0.1) concatenated with ECC outputs) — https://github.com/enveda/ccs-prediction
- **Python** (Runs the training/verification scripts that exercise the implemented module) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --smiles-column-name "smiles" --adduct-column-name "adduct" --ccs-column-name "ccs" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- A forward pass on sample METLIN-CCS or CCSBase molecular graphs completes without shape errors through the ECC layers, the concatenated representation, and the eight 384-neuron FC layers
- The GNN module matches the SigmaCCS original repository (zmzhang/SigmaCCS) structure — three ECC layers built from node feature, adjacency and edge feature matrices
- The parallel branch has exactly four fully connected layers with 256, 512, 512 and 256 neurons and dropout 0.1
- With 3D coordinates supplied via --coordinates-present, the pipeline uses them; otherwise it generates 3D coordinates from SMILES as SigmaCCS does
- Training via scripts/train-test.py with --dropout-rate 0.1 --epochs 400 produces a saved .h5 model and a parameter JSON file

## Limitations

- High accuracy of SigmaCCS trained on CCSBase does not generalize to other chemical spaces; Mol2CCS only partially improves generalizability via additional features
- Models must handle adduct-specific graph construction ([M+H]+, [M+Na]+, [M-H]-) and instrument-type variation, which the article notes likely introduced inaccuracies in training CCS values
- If 3D coordinates are not provided, the model generates them from SMILES, introducing conformer-generation variability (ETKDG seed/rotations affect results)

## Evidence

- [intro] RDKit graph construction for ECC layers: "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
- [discussion] Mol2CCS architecture extension rationale: "we demonstrate how an extension of the architecture of GNN (Mol2CCS) including other additional features improves the generalizability of SigmaCCS"
- [full_text] Parallel FC module specification: [evidence span withheld — anchor status: not_found]
- [readme] Graph conversion requirement from SigmaCCS README: [evidence span withheld — anchor status: external_doc]
