---
name: cross-database-benchmarking
description: Use when evaluating CCS prediction models on CCSBase and/or METLIN-CCS
  data — same-database, cross-database (train-metlin-test-ccsbase), or combined-dataset
  splits — to quantify generalizability via R2, RMSE, and MRE.
when_to_use_negative:
- The goal is predicting CCS for new compounds with an already-trained model rather
  than benchmarking model performance against reference databases
- The CCS data comes from a different database or adduct set outside CCSBase/METLIN-CCS
  (e.g. non-[M+H]+, [M+Na]+, [M-H]- adducts), so the trained models and splits are
  not applicable
- The focus is confidence-model filtering or applicability-domain analysis rather
  than head-to-head model benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3335
tools:
- name: SigmaCCS
  role: GNN model being benchmarked for CCS prediction across database configurations
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: Graph convolutional network model being benchmarked for CCS prediction
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Extension of SigmaCCS with additional features (fingerprints, descriptors,
    molecule types) benchmarked for improved generalizability
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit
  role: Used to read SMILES and generate/validate molecular representations and 3D
    coordinates for model input
- name: poetry
  role: Environment and script runner for the train-test.py benchmarking pipeline
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:923354de8512911bd3017d9c063e3f63ced6c4a0de2a676bc2689e0334ad35d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmzhang/SigmaCCS
  tools:
  - GraphCCS
  - Mol2CCS
  - Python
  - RDKit
  - SigmaCCS
  - poetry
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-database-benchmarking

## Summary

Benchmark GNN-based CCS prediction models (SigmaCCS, GraphCCS, Mol2CCS) under matched training/evaluation configurations on the CCSBase (v1.3) and METLIN-CCS databases to quantify in-domain accuracy and cross-database generalizability.

## When to use

Use when evaluating CCS prediction models on CCSBase and/or METLIN-CCS data — same-database, cross-database (train-metlin-test-ccsbase), or combined-dataset splits — to quantify generalizability via R2, RMSE, and MRE.

## When NOT to use

- The goal is predicting CCS for new compounds with an already-trained model rather than benchmarking model performance against reference databases
- The CCS data comes from a different database or adduct set outside CCSBase/METLIN-CCS (e.g. non-[M+H]+, [M+Na]+, [M-H]- adducts), so the trained models and splits are not applicable
- The focus is confidence-model filtering or applicability-domain analysis rather than head-to-head model benchmarking

## Inputs

- Raw CCSBase (v1.3) and METLIN-CCS downloads (excel/csv, obtained per their licenses from ccsbase.net and metlin.scripps.edu)
- Processed 80/20 train/test parquet splits with coordinates, smiles, adduct, and ccs columns (e.g. ccs-prediction/metlin_train_3d.parquet, ccs-prediction/ccsbase_3d.parquet)
- Optionally, pre-computed prediction files from Zenodo (10.5281/zenodo.11199061) unzipped into the data directory

## Outputs

- Trained model files (model/<prefix>.h5) and parameter JSON files (parameter/parameter-<prefix>.json)
- Per-run test-set prediction files named by prefix (e.g. train-metlin-test-ccsbase)
- Per-database/per-model metrics table (metrics_table.csv or metrics_table.tsv) with R2, RMSE, and MRE columns

## How to apply

Process the raw CCSBase (v1.3, 16,989 CCS values from 6,744 molecules) and METLIN-CCS (27,000+ unique structures) downloads with the notebooks/data_processing notebooks, especially 2_data_splits.ipynb, to produce 80/20 train/test parquet splits carrying coordinates, smiles, adduct, and ccs columns. Train each of the three models (SigmaCCS, GraphCCS, Mol2CCS) via the Makefile or scripts/train-test.py with dropout-rate 0.1 and 400 epochs, running each configuration: same-database (e.g. make train-metlin-test-metlin), cross-database (e.g. train-metlin-test-ccsbase), and combined CCSBase+METLIN. Evaluate each trained model on its test split and compute R2, RMSE, and mean relative error (MRE) per model per configuration, assembling them into metrics_table.csv/tsv. Rationale: same-database training yields near-ceiling performance (R2 slightly below 0.986, MRE ~1.51–1.57% on CCSBase), whereas cross-database evaluation exposes the decline in structurally novel chemical space, which is the core generalizability signal of the benchmark.

## Related tools

- **SigmaCCS** (GNN model being benchmarked for CCS prediction across database configurations) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (Graph convolutional network model being benchmarked for CCS prediction) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Extension of SigmaCCS with additional features (fingerprints, descriptors, molecule types) benchmarked for improved generalizability) — https://github.com/enveda/ccs-prediction
- **RDKit** (Used to read SMILES and generate/validate molecular representations and 3D coordinates for model input)
- **poetry** (Environment and script runner for the train-test.py benchmarking pipeline) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- Same-database CCSBase runs reproduce R2 slightly below 0.986 with MRE ~1.55–1.57% for GraphCCS/SigmaCCS and R2 = 0.985, MRE = 1.51% for Mol2CCS
- METLIN-CCS-trained models reach R2 around 0.9 on their own test split
- Combined CCSBase+METLIN runs give R2 close to 0.95, RMSE close to 6, and MRE between 1.39% and 1.71%, matching Fig. 4 of the manuscript
- Cross-database runs (train-metlin-test-ccsbase and reverse) show a marked performance drop relative to same-database runs, consistent with the reported lack of generalization
- Metrics table contains one row per model per configuration with R2, RMSE, and MRE populated and no missing test predictions

## Limitations

- High same-database accuracy does not generalize: performance significantly declines on structurally novel regions outside the training chemical space
- Evaluation is restricted in scope mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures)
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values, motivating the filtering strategies applied

## Evidence

- [methods] makefile train command: "See the commands in the `Makefile` to train the models. Run them as `make train-metlin-test-metlin`"
- [discussion] generalization failure: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
- [discussion] cross-database direction: "We observed similar results when training on METLIN-CCS and evaluating on CCSBase"
- [intro] benchmark scope: "In this work we benchmark the state-of-the-art models on the new METLIN-CCS database and assess their generalizability"
- [readme] readme training invocation: "poetry run python scripts/train-test.py \
	--prefix "train-metlin-test-ccsbase" \
	--train-input-file "ccs-prediction/metlin_train_3d.parquet""
- [readme] data splits provenance: "--train-input-file is the training set (see notebooks/data_processing/2_data_splits.ipynb for details on the format)"
