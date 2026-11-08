---
name: adduct-aware-ccs-prediction-evaluation
description: Use when you need to test whether a CCS prediction model trained on one
  ion-mobility database (e.g. CCSBase v1.3, 16,989 values / 6,744 molecules) generalizes
  to a structurally different database such as METLIN-CCS (>27,000 unique synthetic
  small-molecule structures).
when_to_use_negative:
- Your training and test compounds come from the same database/chemical space — the
  skill is specifically about cross-database generalization; within-distribution accuracy
  was already reported as high for all models
- You aim to improve generalization itself (e.g. via fingerprints, descriptors, instrument
  type, or confidence-model filtering) rather than quantify the drop; Mol2CCS extensions
  are a separate skill
- Your data is not ion-mobility CCS paired with SMILES/adduct annotations (e.g. retention
  time or MS/MS spectra prediction tasks)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Runtime for the train-test.py cross-database training/evaluation script, invoked
    via poetry
  repo: https://github.com/enveda/ccs-prediction
- name: poetry
  role: Dependency manager used to install the ccs-prediction environment (poetry
    install) and run scripts (poetry run python scripts/train-test.py)
  repo: https://github.com/enveda/ccs-prediction
- name: SigmaCCS
  role: GNN baseline (TensorFlow/spektral-based) trained and cross-evaluated on CCSBase
    and METLIN-CCS; achieves ~0.8 R2 when trained on CCSBase and tested on METLIN-CCS
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: GCN baseline (pytorch/DGL-based) cross-evaluated between databases; shows
    the largest generalization drop (R2 ≈ 0.36 for CCSBase→METLIN-CCS)
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Extension of SigmaCCS with additional features (fingerprints, descriptors,
    molecule/instrument type) used as the generalization-improved comparator in the
    benchmark
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit (v2023.09.5)
  role: Reads SMILES and generates/validates 3D coordinates of molecules used as graph-model
    input features
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:ded89469385b1cb4946a79a5480b896b1352250b78b59ed312c71ec48fb6971f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - GraphCCS
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - SigmaCCS
  - poetry
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# adduct-aware-ccs-prediction-evaluation

## Summary

Benchmark graph neural network CCS prediction models (SigmaCCS, GraphCCS, Mol2CCS) in a cross-database setting: train on one CCS database (CCSBase v1.3 or METLIN-CCS) and evaluate on the structurally different one, quantifying generalization via R2, MAE, and MRE.

## When to use

Use when you need to test whether a CCS prediction model trained on one ion-mobility database (e.g. CCSBase v1.3, 16,989 values / 6,744 molecules) generalizes to a structurally different database such as METLIN-CCS (>27,000 unique synthetic small-molecule structures).

## When NOT to use

- Your training and test compounds come from the same database/chemical space — the skill is specifically about cross-database generalization; within-distribution accuracy was already reported as high for all models
- You aim to improve generalization itself (e.g. via fingerprints, descriptors, instrument type, or confidence-model filtering) rather than quantify the drop; Mol2CCS extensions are a separate skill
- Your data is not ion-mobility CCS paired with SMILES/adduct annotations (e.g. retention time or MS/MS spectra prediction tasks)

## Inputs

- Formatted CCSBase v1.3 training/test parquet files (e.g. ccsbase_3d.parquet) with smiles, adduct, coordinates, and ccs columns
- Formatted METLIN-CCS training/test parquet files (e.g. metlin_train_3d.parquet) with the same columns
- Raw CCSBase (from https://ccsbase.net/query) and METLIN-CCS (https://metlin.scripps.edu/) downloads, preprocessed via the data_processing notebooks
- Model hyperparameter JSON (e.g. parameter/parameter-train-metlin-test-metlin.json)

## Outputs

- Trained model files (.h5) and parameter files per train/test combination (e.g. model/train-metlin-test-metlin.h5)
- Prediction files for each test set, named by prefix (e.g. train-metlin-test-ccsbase)
- Cross-database evaluation metrics table with R2, MAE, and MRE per model and per train→test direction, reproducing Fig. 3 (e.g. R2 ≈ 0.36 for GraphCCS, ≈ 0.8 for SigmaCCS, ≈ 0.84 for Mol2CCS for CCSBase→METLIN-CCS)
- Console log files (e.g. train-metlin-test-ccsbase.out)

## How to apply

First run the data_processing notebooks (notebooks/data_processing, including 2_data_splits.ipynb) to convert the raw METLIN-CCS and CCSBase excel/csv downloads into formatted training/test parquet files containing smiles, adduct, coordinates, and ccs columns. For each model (SigmaCCS, GraphCCS, Mol2CCS), train on one database using scripts/train-test.py with explicit column arguments (--smiles-column-name, --adduct-column-name, --ccs-column-name, --coordinates-column-name and --coordinates-present so 3D coordinates are used rather than regenerated from SMILES), dropout rate 0.1 and 400 epochs, using a prefix like 'train-ccsbase-test-metlin' to name output prediction files. Evaluate the trained model on the held-out other-database test set and record R2, MAE, and MRE; then repeat with the database roles reversed (e.g. 'train-metlin-test-ccsbase'). A correct reproduction matches the Fig. 3 cross-database drops: when trained on CCSBase and evaluated on METLIN-CCS, R2 falls to 0.36 (GraphCCS), ~0.8 (SigmaCCS), and 0.84 (Mol2CCS); expect similar lack of generalizability in the reverse direction. Optionally aggregate all four train/test combinations into a single metrics table.

## Related tools

- **Python** (Runtime for the train-test.py cross-database training/evaluation script, invoked via poetry) — https://github.com/enveda/ccs-prediction
- **poetry** (Dependency manager used to install the ccs-prediction environment (poetry install) and run scripts (poetry run python scripts/train-test.py)) — https://github.com/enveda/ccs-prediction
- **SigmaCCS** (GNN baseline (TensorFlow/spektral-based) trained and cross-evaluated on CCSBase and METLIN-CCS; achieves ~0.8 R2 when trained on CCSBase and tested on METLIN-CCS) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (GCN baseline (pytorch/DGL-based) cross-evaluated between databases; shows the largest generalization drop (R2 ≈ 0.36 for CCSBase→METLIN-CCS)) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Extension of SigmaCCS with additional features (fingerprints, descriptors, molecule/instrument type) used as the generalization-improved comparator in the benchmark) — https://github.com/enveda/ccs-prediction
- **RDKit (v2023.09.5)** (Reads SMILES and generates/validates 3D coordinates of molecules used as graph-model input features)

## Examples

```
poetry run python scripts/train-test.py --prefix "train-ccsbase-test-metlin" --train-input-file "ccs-prediction/ccsbase_3d.parquet" --test-input-file "ccs-prediction/metlin_3d.parquet" --smiles-column-name "smiles" --adduct-column-name "adduct" --ccs-column-name "ccs" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- CCSBase→METLIN-CCS R2 values match Fig. 3: GraphCCS ≈ 0.36, SigmaCCS ≈ 0.8, Mol2CCS ≈ 0.84
- Within-database (train and test on same database) performance is substantially higher than the cross-database drop, confirming the gap is a generalization effect
- Reverse direction (METLIN-CCS→CCSBase) also shows degraded metrics, consistent with the paper's 'similar results when training on METLIN-CCS and evaluating on CCSBase'
- All runs report R2, MAE, and MRE per model and direction; the four train/test combinations are aggregated into one metrics table
- Prediction output files are generated with the correct prefix (e.g. train-metlin-test-ccsbase) and the model/parameter files exist (e.g. model/train-metlin-test-metlin.h5)

## Limitations

- Evaluation is restricted in scope mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures)
- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS training values
- Cross-database results reflect lack of generalizability: the high accuracy reported when models were trained on CCSBase does not generalize to other chemical spaces
- High-quality experimental CCS references number only in the low thousands, constraining training data

## Evidence

- [results] cross-database performance targets: [evidence span withheld — anchor status: not_found]
- [discussion] benchmark scope: "we benchmarked two GNNs (i.e., SigmaCCS and GraphCCS) and Mol2CCS, an adaptation of SigmaCCS, on METLIN-CCS and CCSBase"
- [discussion] lack of generalizability finding: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
- [discussion] reverse direction: "We observed similar results when training on METLIN-CCS and evaluating on CCSBase"
- [readme] training invocation: "poetry run python scripts/train-test.py \
	--prefix "train-metlin-test-ccsbase" \
	--train-input-file "ccs-prediction/metlin_train_3d.parquet""
- [readme] data splits provenance: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
