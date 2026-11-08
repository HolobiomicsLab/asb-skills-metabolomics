---
name: graph-neural-network-model-evaluation
description: Use when you need to train and compare multiple GNN regression models
  on the same train/test parquet splits and produce a per-model metrics table (R2,
  RMSE, MRE) to judge relative performance.
when_to_use_negative:
- When the goal is to assess cross-database generalizability (e.g. train on METLIN-CCS
  and test on CCSBase) — the paper shows within-space accuracy (R2 ~0.95) does not
  transfer to structurally novel chemical spaces
- When input data has not been standardized and split by the data_processing notebooks
  — raw CCSBase/METLIN downloads need filtering and formatting first
- When predicting CCS for a single compound rather than benchmarking models — use
  the individual prediction workflows (Model_prediction in SigmaCCS, GraphCCS prediction)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Runtime for the train-test.py benchmarking script
- name: poetry
  role: Dependency management and invocation of training scripts in the enveda/ccs-prediction
    environment
  repo: https://github.com/enveda/ccs-prediction
- name: SigmaCCS
  role: GNN model benchmarked on combined CCSBase + METLIN-CCS data
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: GCN model benchmarked on combined CCSBase + METLIN-CCS data; best performer
    on the combined dataset
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Extension of SigmaCCS with fingerprints, descriptors, and molecule/instrument-type
    features, benchmarked alongside the two GNNs
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:fc3ffdea963ee0bc0f749d8c732c0dff4ac51c6e6db9b2da08bfdeabe8c2b6ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - DeepCCS
  - Graph Neural Networks
  - GraphCCS
  - Mol2CCS
  - Python
  - RDKit
  - SigmaCCS
  - Spektral
  - TensorFlow
  - poetry
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# graph-neural-network-model-evaluation

## Summary

Benchmarks state-of-the-art graph neural network CCS prediction models (SigmaCCS, GraphCCS, Mol2CCS) by training them on combined CCSBase + METLIN-CCS splits and collecting per-model R2, RMSE, and MRE on a held-out test set.

## When to use

Use when you need to train and compare multiple GNN regression models on the same train/test parquet splits and produce a per-model metrics table (R2, RMSE, MRE) to judge relative performance.

## When NOT to use

- When the goal is to assess cross-database generalizability (e.g. train on METLIN-CCS and test on CCSBase) — the paper shows within-space accuracy (R2 ~0.95) does not transfer to structurally novel chemical spaces
- When input data has not been standardized and split by the data_processing notebooks — raw CCSBase/METLIN downloads need filtering and formatting first
- When predicting CCS for a single compound rather than benchmarking models — use the individual prediction workflows (Model_prediction in SigmaCCS, GraphCCS prediction)

## Inputs

- Training-set parquet (e.g. metlin_train_3d.parquet) with 3D coordinates, smiles, adduct, and ccs columns
- Test-set parquet (e.g. ccsbase_3d.parquet) in the same column format
- Model hyperparameters (dropout-rate 0.1, epochs 400)

## Outputs

- Trained model file (.h5) per model
- JSON parameter file (e.g. parameter-train-metlin-test-metlin.json) per model
- Test-set prediction files generated via the --prefix argument
- metrics_table.tsv with one row per model and columns R2, RMSE, MRE

## How to apply

Load the combined CCSBase + METLIN-CCS train and test parquet splits produced by the data_processing notebooks, pointing train-test.py to the coordinates, smiles, adduct, and ccs columns. Train each model (SigmaCCS, GraphCCS, Mol2CCS) via `poetry run python scripts/train-test.py` with dropout-rate 0.1 and 400 epochs, saving the .h5 model file and the JSON parameter file per run. Run predictions on the test split for each trained model and collect per-model R2, RMSE, and MRE. Assemble the metrics into a single metrics_table.tsv with one row per model; in the paper's combined-dataset setting all three models reach R2 close to 0.95, RMSE close to 6, and MRE between 1.39% and 1.71%, with GraphCCS the best.

## Related tools

- **Python** (Runtime for the train-test.py benchmarking script)
- **poetry** (Dependency management and invocation of training scripts in the enveda/ccs-prediction environment) — https://github.com/enveda/ccs-prediction
- **SigmaCCS** (GNN model benchmarked on combined CCSBase + METLIN-CCS data) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (GCN model benchmarked on combined CCSBase + METLIN-CCS data; best performer on the combined dataset) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Extension of SigmaCCS with fingerprints, descriptors, and molecule/instrument-type features, benchmarked alongside the two GNNs) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- Each model's test-set R2 on the combined CCSBase + METLIN-CCS split is close to 0.95
- Each model's test-set RMSE is close to 6
- Each model's MRE falls between 1.39% and 1.71%, matching the values reported in Fig. 4 of the manuscript
- Each training run produced both a .h5 model file and a JSON parameter file at the paths given via --model-output-file and --parameter-path
- metrics_table.tsv has exactly one row per model with populated R2, RMSE, and MRE columns

## Limitations

- High accuracy within chemical spaces similar to the training environment does not generalize to structurally novel regions — e.g. models trained on CCSBase do not transfer to METLIN-CCS and vice versa
- Evaluation is restricted mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures)
- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS training values

## Evidence

- [results] combined-dataset benchmark metrics: [evidence span withheld — anchor status: not_found]
- [readme] training invocation: "poetry run python scripts/train-test.py \
	--prefix "train-metlin-test-ccsbase" \
	--train-input-file "ccs-prediction/metlin_train_3d.parquet""
- [readme] hyperparameters: "--dropout-rate 0.1 \
	--epochs 400"
- [discussion] generalizability caveat: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
- [discussion] scope limitation: "its restriction in scope, as it mainly focuses on small molecules and specific regions of the chemical space (e.g., METLIN-CCS is based on synthetic structures)"
