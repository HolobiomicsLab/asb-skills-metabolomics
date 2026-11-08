---
name: gnn-ccs-regression-training
description: Use when you have SMILES, adduct, and CCS value columns (optionally 3D
  coordinates) split into train/test parquet files and need to train and benchmark
  GNN CCS regression models like SigmaCCS, GraphCCS, or Mol2CCS.
when_to_use_negative:
- Evaluating cross-dataset generalizability (e.g. train on METLIN-CCS, test on CCSBase)
  rather than combined-dataset training — performance declines sharply outside the
  training chemical space and the combined-training metrics do not apply
- You only need prediction with a pre-trained model — use the Model_prediction / prediction
  scripts instead of retraining
- Input data lacks SMILES/adduct annotations or has not been processed into the parquet
  schema expected by train-test.py — run the data_processing notebooks first
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0154
tools:
- name: poetry
  role: Manages the development environment and invokes the training script (poetry
    run python scripts/train-test.py)
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Runs the train-test.py script that trains and evaluates the three models
  repo: https://github.com/enveda/ccs-prediction
- name: SigmaCCS
  role: One of the benchmarked GNN architectures; structure-included graph merged
    with adduct method for CCS prediction
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: One of the benchmarked GCN architectures; end-to-end learning on adduct graphs
    generated from SMILES plus adduct type
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Adaptation of SigmaCCS extended with additional features (molecular fingerprints,
    descriptors, molecule/instrument types) for improved generalizability
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit
  role: Reads SMILES representations and generates/optimizes 3D conformations (ETKDGv3
    + MMFF) for graph construction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:2b36a2fc2136cd463b93fc6575d9f3ec16b3f3ca846964e7c7f40c8fc806a609
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gnn-ccs-regression-training

## Summary

Training graph neural network regression models (SigmaCCS, GraphCCS, Mol2CCS) on combined CCSBase + METLIN-CCS collision cross section data to predict CCS values from 3D molecular coordinates, SMILES, and adduct type, then evaluating with R2, RMSE, and MRE.

## When to use

Use when you have SMILES, adduct, and CCS value columns (optionally 3D coordinates) split into train/test parquet files and need to train and benchmark GNN CCS regression models like SigmaCCS, GraphCCS, or Mol2CCS.

## When NOT to use

- Evaluating cross-dataset generalizability (e.g. train on METLIN-CCS, test on CCSBase) rather than combined-dataset training — performance declines sharply outside the training chemical space and the combined-training metrics do not apply
- You only need prediction with a pre-trained model — use the Model_prediction / prediction scripts instead of retraining
- Input data lacks SMILES/adduct annotations or has not been processed into the parquet schema expected by train-test.py — run the data_processing notebooks first

## Inputs

- Training split parquet file (e.g. ccs-prediction/metlin_train_3d.parquet) with columns: coordinates (3D), smiles, adduct, ccs
- Test split parquet file (e.g. ccs-prediction/ccsbase_3d.parquet) with the same column schema
- Raw CCSBase (https://ccsbase.net/query) and METLIN-CCS (https://metlin.scripps.edu/) downloads formatted via the data_processing notebooks

## Outputs

- Trained model file (.h5) per model (SigmaCCS, GraphCCS, Mol2CCS)
- JSON parameter file per model (e.g. parameter/parameter-train-metlin-test-metlin.json)
- Per-model test-set predictions on the test split, named by prefix
- Metrics table (metrics_table.tsv) with one row per model and columns R2, RMSE, MRE

## How to apply

First run the data_processing notebooks to standardize CCSBase (v1.3) and METLIN-CCS into parquet files with explicit columns for 3D coordinates, SMILES, adduct, and CCS, and produce train-test splits (notebooks/data_processing/2_data_splits.ipynb). Then train each of the three models by invoking scripts/train-test.py via poetry, passing the train and test parquet paths, --coordinates-column-name 'coordinates' with --coordinates-present, --smiles-column-name 'smiles', --adduct-column-name 'adduct', --ccs-column-name 'ccs', --dropout-rate 0.1, and --epochs 400; each run saves a model .h5 file and a JSON parameter file. The prefix argument (e.g. 'train-metlin-test-ccsbase') names the prediction outputs for the test set. Run predictions on the test split for each trained model and collect per-model R2, RMSE, and MRE. Rationale: the paper benchmarks two GNNs (SigmaCCS and GraphCCS) plus Mol2CCS, an adaptation of SigmaCCS with additional features (fingerprints, descriptors, molecule/instrument types), and combined-dataset training is the reference setting in which all three reach R2 close to 0.95 with GraphCCS the best.

## Related tools

- **poetry** (Manages the development environment and invokes the training script (poetry run python scripts/train-test.py)) — https://github.com/enveda/ccs-prediction
- **Python** (Runs the train-test.py script that trains and evaluates the three models) — https://github.com/enveda/ccs-prediction
- **SigmaCCS** (One of the benchmarked GNN architectures; structure-included graph merged with adduct method for CCS prediction) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (One of the benchmarked GCN architectures; end-to-end learning on adduct graphs generated from SMILES plus adduct type) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Adaptation of SigmaCCS extended with additional features (molecular fingerprints, descriptors, molecule/instrument types) for improved generalizability) — https://github.com/enveda/ccs-prediction
- **RDKit** (Reads SMILES representations and generates/optimizes 3D conformations (ETKDGv3 + MMFF) for graph construction)

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --coordinates-column-name "coordinates" --coordinates-present --smiles-column-name "smiles" --adduct-column-name "adduct" --ccs-column-name "ccs" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- Each model's combined-dataset test-set R2 is close to 0.95, RMSE close to 6, and MRE falls between 1.39% and 1.71%, matching Fig. 4 of the manuscript
- metrics_table.tsv contains exactly one row per model with columns R2, RMSE, and MRE
- Each training run produced both the .h5 model file and the JSON parameter file at the paths given to --model-output-file and --parameter-path
- Test-split prediction outputs exist for each model under the --prefix naming scheme
- Input parquet files contain the expected columns (coordinates, smiles, adduct, ccs) when --coordinates-present is set

## Limitations

- High accuracy within similar chemical spaces does not generalize to structurally novel regions; cross-dataset evaluation (e.g. train METLIN-CCS, test CCSBase) shows significantly declined performance
- Combined-dataset evaluation mostly covers small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures)
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values

## Evidence

- [readme] poetry run python scripts/train-test.py \
	--prefix "train-metlin-test-ccsbase": "poetry run python scripts/train-test.py \
	--prefix "train-metlin-test-ccsbase" \
	--train-input-file "ccs-prediction/metlin_train_3d.parquet""
- [readme] --dropout-rate 0.1 \
	--epochs 400: "--dropout-rate 0.1 \
	--epochs 400 \
	> train-metlin-test-ccsbase.out 2>&1"
- [discussion] we benchmarked two GNNs (i.e., SigmaCCS and GraphCCS) and Mol2CCS, an adaptation of SigmaCCS, on METLIN-CCS and CCSBase: "we benchmarked two GNNs (i.e., SigmaCCS and GraphCCS) and Mol2CCS, an adaptation of SigmaCCS, on METLIN-CCS and CCSBase"
- [methods] data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits.: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
- [results] When trained and evaluated on the combined CCSBase + METLIN-CCS dataset, all three models reach high performance: R2 close to 0.95: [evidence span withheld — anchor status: not_found]
