---
name: train-test-split-evaluation
description: 'Use when you need a same-database accuracy baseline for CCS prediction
  models: you have processed 80/20 train/test splits of CCSBase or METLIN-CCS (parquet
  files with SMILES, adduct, and CCS columns) and want R2/MRE metrics per model.'
when_to_use_negative:
- When the goal is cross-database generalizability testing (train on one database,
  e.g. METLIN-CCS, and test on another, e.g. CCSBase) — this skill is the same-database
  baseline, not the transfer evaluation
- When input molecules are outside the small-molecule synthetic-structure scope of
  METLIN-CCS or the CCSBase chemical space — the article notes its evaluation is restricted
  mainly to small molecules and specific regions of chemical space
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Runtime for the train-test.py training/evaluation script executed via poetry
  repo: https://github.com/enveda/ccs-prediction
- name: SigmaCCS
  role: One of the three benchmarked GNN models trained and tested on the same-database
    80/20 splits
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: One of the three benchmarked GNN models trained and tested on the same-database
    80/20 splits
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Extension of SigmaCCS with additional features (e.g. instrument type), benchmarked
    alongside the two original GNNs
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit
  role: Reads SMILES representations and generates 3D coordinates/conformations used
    as model inputs
  repo: https://github.com/enveda/ccs-prediction
- name: poetry
  role: Dependency management and entry point for running the training/evaluation
    scripts
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:3a0e12a16a33efae30915c8d6f6f2c0cdfab1df7897790cfd5af6f1141e05619
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - GraphCCS
  - Mol2CCS
  - Python
  - RDKit
  - SigmaCCS
  - poetry
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Train-test split evaluation of CCS prediction models

## Summary

Trains SigmaCCS, GraphCCS, and Mol2CCS on the 80% training split of a CCS database (CCSBase v1.3 or METLIN-CCS) and evaluates each model on the same database's held-out 20% test split, reporting R2 and mean relative error (MRE) to establish the within-chemical-space accuracy baseline before any cross-database generalizability testing.

## When to use

Use when you need a same-database accuracy baseline for CCS prediction models: you have processed 80/20 train/test splits of CCSBase or METLIN-CCS (parquet files with SMILES, adduct, and CCS columns) and want R2/MRE metrics per model.

## When NOT to use

- When the goal is cross-database generalizability testing (train on one database, e.g. METLIN-CCS, and test on another, e.g. CCSBase) — this skill is the same-database baseline, not the transfer evaluation
- When input molecules are outside the small-molecule synthetic-structure scope of METLIN-CCS or the CCSBase chemical space — the article notes its evaluation is restricted mainly to small molecules and specific regions of chemical space

## Inputs

- Processed 80/20 train/test split parquet files for CCSBase v1.3 (e.g. ccsbase_train_3d.parquet / ccsbase_3d.parquet) and METLIN-CCS (e.g. metlin_train_3d.parquet)
- Column names for SMILES, adduct, CCS, and optional 3D coordinates within the parquet files
- Model definitions for SigmaCCS, GraphCCS, and Mol2CCS (from zmzhang/SigmaCCS, tingxiecsu/GraphCCS, and the enveda/ccs-prediction adaptation)

## Outputs

- Per-run prediction files for the test set (prefixed, e.g. train-metlin-test-metlin)
- Trained model files (.h5) and parameter JSON files per run (e.g. model/train-metlin-test-metlin.h5, parameter/parameter-train-metlin-test-metlin.json)
- metrics_table.csv with per-database, per-model R2 and MRE

## How to apply

Load the processed 80/20 train/test splits produced by the data_processing notebooks (notebooks/data_processing/2_data_splits.ipynb) for CCSBase and METLIN-CCS. Train each of the three models (SigmaCCS, GraphCCS, Mol2CCS) on the 80% split via the Makefile targets (e.g. `make train-metlin-test-metlin`), which invoke scripts/train-test.py with the training parquet, adduct/SMILES/CCS column names, dropout rate 0.1, and 400 epochs. Evaluate each trained model on the same database's 20% test split, producing prediction files prefixed per run (e.g. train-metlin-test-metlin). Compute R2 and MRE per model per database from the test predictions, then assemble the per-database, per-model metrics into a table (metrics_table.csv). The decision point is that these same-database numbers are the ceiling benchmark: R2 slightly below 0.986 with MRE around 1.55-1.57% for CCSBase-trained models (GraphCCS, SigmaCCS) and Mol2CCS at R2 = 0.985, MRE = 1.51%.

## Related tools

- **Python** (Runtime for the train-test.py training/evaluation script executed via poetry) — https://github.com/enveda/ccs-prediction
- **SigmaCCS** (One of the three benchmarked GNN models trained and tested on the same-database 80/20 splits) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (One of the three benchmarked GNN models trained and tested on the same-database 80/20 splits) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Extension of SigmaCCS with additional features (e.g. instrument type), benchmarked alongside the two original GNNs) — https://github.com/enveda/ccs-prediction
- **RDKit** (Reads SMILES representations and generates 3D coordinates/conformations used as model inputs) — https://github.com/enveda/ccs-prediction
- **poetry** (Dependency management and entry point for running the training/evaluation scripts) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-metlin" --smiles-column-name "smiles" --adduct-column-name "adduct" --ccs-column-name "ccs" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- Reproduced CCSBase-trained metrics should match reported values: R2 slightly below 0.986 with MRE around 1.55-1.57% for GraphCCS and SigmaCCS, and R2 = 0.985 / MRE = 1.51% for Mol2CCS
- METLIN-CCS-trained models should reach R2 around 0.9 on their own test split
- Prediction files must be generated for every run with the expected prefix (e.g. train-metlin-test-metlin) and cover the full 20% test split
- metrics_table.csv must contain one R2 and MRE entry per model per database with no missing model/database combinations
- Training run must complete the specified 400 epochs with dropout rate 0.1 and produce the model .h5 and parameter JSON files at the configured paths

## Limitations

- Same-database performance is high (R2 ~0.986 on CCSBase) but does not generalize: the original high accuracy of models trained on CCSBase does not transfer to other chemical spaces, so these metrics are a within-distribution ceiling only
- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS values used to train the models
- Evaluation is restricted in scope to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures)

## Evidence

- [abstract] High within-database accuracy benchmark: "our results confirm the high accuracy of these models within chemical spaces similar to their training environments"
- [methods] Data split provenance: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
- [readme] Model training entry point: "See the commands in the `Makefile` to train the models. Run them as `make train-metlin-test-metlin`"
- [discussion] Cross-database contrast motivating the baseline: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
