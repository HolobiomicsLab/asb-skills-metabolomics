---
name: gnn-ccs-cross-database-training
description: Use when you need to test whether a GNN-based CCS prediction model generalizes
  across chemically distinct reference databases, e.g. train on CCSBase and test on
  METLIN-CCS or vice versa.
when_to_use_negative:
- You only need within-database accuracy (train-metlin-test-metlin style splits);
  the cross-database direction measures generalization, not absolute model quality
- Your molecules fall outside the small, mostly synthetic structures covered by METLIN-CCS
  and CCSBase — the paper's evaluation is explicitly restricted in scope to small
  molecules and specific regions of chemical space
- You lack one of the two databases formatted with matching adduct, SMILES, coordinates,
  and CCS columns required by train-test.py
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: SigmaCCS
  role: GNN baseline trained on one database and evaluated on the other
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: GCN baseline (PyTorch/DGL) trained and cross-evaluated across CCSBase and
    METLIN-CCS
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Extended SigmaCCS architecture adding fingerprints, descriptors, and instrument
    type for improved generalization
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit (v2023.09.5)
  role: Reads SMILES and generates 3D conformations (ETKDGv3 + MMFF) when coordinates
    are not provided
- name: poetry
  role: Installs the enveda/ccs-prediction environment to run scripts/train-test.py
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:fb3f02c86810ef4371987cd075f7246f31ae6be4d7eaaeb0302718ebc1823aca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GraphCCS
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - SigmaCCS
  - poetry
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gnn-ccs-cross-database-training

## Summary

Trains SigmaCCS, GraphCCS, or Mol2CCS GNN models on one CCS database (CCSBase v1.3 or METLIN-CCS) and evaluates them on the structurally different held-out database to quantify generalization, reproducing the Fig. 3 performance drops reported in Engler Hart et al. 2024.

## When to use

Use when you need to test whether a GNN-based CCS prediction model generalizes across chemically distinct reference databases, e.g. train on CCSBase and test on METLIN-CCS or vice versa.

## When NOT to use

- You only need within-database accuracy (train-metlin-test-metlin style splits); the cross-database direction measures generalization, not absolute model quality
- Your molecules fall outside the small, mostly synthetic structures covered by METLIN-CCS and CCSBase — the paper's evaluation is explicitly restricted in scope to small molecules and specific regions of chemical space
- You lack one of the two databases formatted with matching adduct, SMILES, coordinates, and CCS columns required by train-test.py

## Inputs

- Formatted parquet training split from one CCS database (e.g. ccs-prediction/metlin_train_3d.parquet)
- Formatted parquet test split from the other database (e.g. ccs-prediction/ccsbase_3d.parquet)
- Per-molecule columns: SMILES, adduct, CCS value, and 3D coordinates (or SMILES-only if coordinates absent)
- Raw CCSBase (https://ccsbase.net/query) and METLIN (https://metlin.scripps.edu/) downloads formatted via the data_processing notebooks

## Outputs

- Trained .h5 model file and JSON parameter file per train/test direction
- Prediction files for the test set keyed by the --prefix argument (e.g. train-ccsbase-test-metlin)
- Cross-database metrics table (R2, MAE, MRE per model per direction) reproducing the Fig. 3 comparison

## How to apply

First run the notebooks/data_processing notebooks (1 database ingestion, 2_data_splits.ipynb) to produce parquet training and test files for each database, or download the precomputed predictions from Zenodo (10.5281/zenodo.11199061) into the data directory. Then call scripts/train-test.py with the --prefix set to the train/test direction (e.g. 'train-ccsbase-test-metlin'), passing --train-input-file, --test-input-file, --parameter-path, --model-output-file, and the column arguments --coordinates-column-name, --smiles-column-name, --adduct-column-name, --ccs-column-name, plus --coordinates-present when 3D coordinates exist, --dropout-rate 0.1 and --epochs 400. Evaluate each trained model on the other database's test set and record R2, MAE, and MRE per model, then reverse the train/test roles and repeat. Compare the aggregated metrics against the expected degradation: for CCSBase→METLIN-CCS, R2 ≈ 0.36 (GraphCCS), ≈ 0.8 (SigmaCCS), ≈ 0.84 (Mol2CCS), confirming that within-database accuracy does not transfer across chemical spaces.

## Related tools

- **SigmaCCS** (GNN baseline trained on one database and evaluated on the other) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (GCN baseline (PyTorch/DGL) trained and cross-evaluated across CCSBase and METLIN-CCS) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Extended SigmaCCS architecture adding fingerprints, descriptors, and instrument type for improved generalization) — https://github.com/enveda/ccs-prediction
- **RDKit (v2023.09.5)** (Reads SMILES and generates 3D conformations (ETKDGv3 + MMFF) when coordinates are not provided)
- **poetry** (Installs the enveda/ccs-prediction environment to run scripts/train-test.py) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-ccsbase-test-metlin" --train-input-file ccs-prediction/ccsbase_3d.parquet --test-input-file ccs-prediction/metlin_train_3d.parquet --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- Reproduced CCSBase→METLIN-CCS R2 values match Fig. 3: GraphCCS ≈ 0.36, SigmaCCS ≈ 0.8, Mol2CCS ≈ 0.84
- Cross-database R2 is dramatically lower than the within-database accuracy reported when models are trained on CCSBase, confirming the generalization drop
- The reversed direction (train METLIN-CCS, test CCSBase) shows a similar lack of generalizability as stated in the discussion
- Metrics table contains R2, MAE, and MRE for all three models in both train/test directions
- Training runs complete 400 epochs with dropout rate 0.1 and produce a .h5 model plus parameter JSON per direction

## Limitations

- Generalization is only partially improved by Mol2CCS's additional features (fingerprints, descriptors, instrument type); performance still declines on structurally novel regions
- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS values used for training
- Evaluation scope is restricted mainly to small molecules and specific chemical-space regions (METLIN-CCS is based on synthetic structures), limiting external validity

## Evidence

- [results] Cross-database R2 drops: [evidence span withheld — anchor status: not_found]
- [readme] Training invocation: "poetry run python scripts/train-test.py \
	--prefix "train-metlin-test-ccsbase" \
	--train-input-file "ccs-prediction/metlin_train_3d.parquet""
- [readme] Training hyperparameters: "--dropout-rate 0.1 \
	--epochs 400 \
	> train-metlin-test-ccsbase.out 2>&1"
- [discussion] Generalization finding: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
- [discussion] Reversed direction: "We observed similar results when training on METLIN-CCS and evaluating on CCSBase"
- [methods] Data processing: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
