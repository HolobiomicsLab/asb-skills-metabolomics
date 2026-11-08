---
name: ccs-graph-neural-network-training
description: 'Use when you need a within-database baseline of GNN-based CCS prediction
  accuracy: you have SMILES + adduct + CCS values from CCSBase (v1.3) or METLIN-CCS
  already split 80/20 and want R2/MRE per model.'
when_to_use_negative:
- You want to measure cross-database generalization — use the train-on-one-database/test-on-other
  workflow (e.g. train-metlin-test-ccsbase) instead, since within-database metrics
  will be optimistically high
- Input data is raw database downloads (Excel/CSV) that have not yet been standardized
  and split by the data_processing notebooks
- Your compounds fall outside the training chemical space — the article shows accuracy
  declines sharply on structurally novel regions, so within-database metrics will
  not reflect real performance
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_2269
tools:
- name: SigmaCCS
  role: GNN model benchmarked for CCS prediction using graph merging with adduct method;
    trained via its Model_train interface within the unified train-test.py harness
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: Graph convolutional network benchmarked for CCS prediction; consumes adduct
    graphs built from SMILES + adduct type
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Adaptation of SigmaCCS extended with fingerprints, descriptors, and molecule/instrument-type
    features; one of the three models trained on the splits
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Runs scripts/train-test.py under poetry with dataset/adduct/SMILES/CCS column
    arguments, dropout-rate 0.1 and epochs 400
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit
  role: Reads SMILES and generates/optimizes 3D conformations (ETKDGv3 + MMFF) used
    as graph coordinates for training
  repo: https://github.com/rdkit/rdkit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:7bdbff850f040f06bcbc3f38a5c192a25704253926db336e965c99ef49d0fb99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - Graph Neural Networks
  - GraphCCS
  - Mol2CCS
  - Python
  - RDKit
  - SigmaCCS
  - poetry
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ccs-graph-neural-network-training

## Summary

Trains and evaluates graph neural network CCS predictors (SigmaCCS, GraphCCS, Mol2CCS) on 80/20 train/test splits of CCSBase or METLIN-CCS, reporting R2 and mean relative error (MRE) within a single chemical space.

## When to use

Use when you need a within-database baseline of GNN-based CCS prediction accuracy: you have SMILES + adduct + CCS values from CCSBase (v1.3) or METLIN-CCS already split 80/20 and want R2/MRE per model.

## When NOT to use

- You want to measure cross-database generalization — use the train-on-one-database/test-on-other workflow (e.g. train-metlin-test-ccsbase) instead, since within-database metrics will be optimistically high
- Input data is raw database downloads (Excel/CSV) that have not yet been standardized and split by the data_processing notebooks
- Your compounds fall outside the training chemical space — the article shows accuracy declines sharply on structurally novel regions, so within-database metrics will not reflect real performance

## Inputs

- Processed 80/20 train/test split parquet files (e.g. metlin_train_3d.parquet, ccsbase_3d.parquet) from notebooks/data_processing/2_data_splits.ipynb
- Per-molecule SMILES strings, adduct type ([M+H]+, [M+Na]+, [M-H]-), and experimental CCS values
- Optional precomputed 3D coordinates column (coordinates); otherwise generated from SMILES via RDKit ETKDGv3

## Outputs

- Trained model files (.h5) and parameter JSON files per run (e.g. model/train-metlin-test-metlin.h5)
- Prediction files for the test split prefixed per run (e.g. train-metlin-test-metlin)
- metrics_table.csv with per-database, per-model R2 and MRE

## How to apply

Load the processed 80/20 train/test parquet splits produced by the data_processing notebooks (2_data_splits.ipynb) for CCSBase and METLIN-CCS. Train each of the three models (SigmaCCS, GraphCCS, Mol2CCS) on the 80% split via a Makefile command such as `make train-metlin-test-metlin`, which invokes scripts/train-test.py with the training parquet, adduct/SMILES/CCS column names, dropout rate 0.1, and 400 epochs. Evaluate each trained model on the same database's 20% test split (train-X-test-X prefix), then compute R2 and mean relative error (MRE) per model per database and assemble them into metrics_table.csv. This same-database setting establishes the ceiling accuracy (R2 slightly below 0.986, MRE ~1.51–1.57% on CCSBase) against which cross-database generalization results are compared.

## Related tools

- **SigmaCCS** (GNN model benchmarked for CCS prediction using graph merging with adduct method; trained via its Model_train interface within the unified train-test.py harness) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (Graph convolutional network benchmarked for CCS prediction; consumes adduct graphs built from SMILES + adduct type) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Adaptation of SigmaCCS extended with fingerprints, descriptors, and molecule/instrument-type features; one of the three models trained on the splits) — https://github.com/enveda/ccs-prediction
- **Python** (Runs scripts/train-test.py under poetry with dataset/adduct/SMILES/CCS column arguments, dropout-rate 0.1 and epochs 400) — https://github.com/enveda/ccs-prediction
- **RDKit** (Reads SMILES and generates/optimizes 3D conformations (ETKDGv3 + MMFF) used as graph coordinates for training) — https://github.com/rdkit/rdkit

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-metlin" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --smiles-column-name "smiles" --adduct-column-name "adduct" --ccs-column-name "ccs" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- On CCSBase-trained models, reproduced R2 should be slightly below 0.986 with MRE around 1.55–1.57% for GraphCCS and SigmaCCS, and R2 ≈ 0.985 with MRE ≈ 1.51% for Mol2CCS
- On METLIN-CCS-trained models, reproduced R2 should be around 0.9
- Each run produces prediction files whose prefix matches the Makefile target name (e.g. train-metlin-test-metlin) and a parameter JSON at the given --parameter-path
- Metrics are computed on the held-out 20% test split only, never on training data
- Same train/test split files (from 2_data_splits.ipynb) are reused across all three models so metrics are comparable

## Limitations

- Same-database performance is a ceiling: the article shows the high accuracy on CCSBase does not generalize to other chemical spaces
- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS training values
- Evaluation is restricted mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures)
- Users must download the raw CCSBase and METLIN databases themselves according to their licenses and format them via the notebooks

## Evidence

- [readme] workflow make command: "See the commands in the `Makefile` to train the models. Run them as `make train-metlin-test-metlin`"
- [results] finding on same-database metrics: [evidence span withheld — anchor status: not_found]
- [discussion] benchmarking scope: "we benchmarked two GNNs (i.e., SigmaCCS and GraphCCS) and Mol2CCS, an adaptation of SigmaCCS, on METLIN-CCS and CCSBase"
- [methods] data splits: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
- [discussion] high accuracy does not generalize: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
