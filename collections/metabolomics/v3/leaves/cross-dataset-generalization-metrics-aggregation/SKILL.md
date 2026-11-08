---
name: cross-dataset-generalization-metrics-aggregation
description: Use when a trained GNN CCS predictor must be evaluated on a structurally
  different database than its training set (e.g. CCSBase→METLIN-CCS or METLIN-CCS→CCSBase)
  and per-model R2/MAE/MRE must be compiled into a cross-database comparison table.
when_to_use_negative:
- When evaluating in-distribution performance (train and test on the same database
  split, e.g. train-metlin-test-metlin) — the skill targets cross-chemical-space generalization,
  not standard held-out accuracy
- When the goal is to improve generalization by adding features (fingerprints, descriptors,
  instrument type in Mol2CCS) rather than to quantify the generalization gap itself
- When the datasets are not structurally distinct CCS databases — aggregating metrics
  across random splits of one database will not reproduce the reported generalization
  drop
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: SigmaCCS
  role: GNN model trained and cross-evaluated; one of the three models whose R2/MAE/MRE
    are aggregated (expected R2 ≈ 0.8 on CCSBase→METLIN-CCS)
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: GNN model trained and cross-evaluated; showed the steepest generalization
    drop (expected R2 ≈ 0.36 on CCSBase→METLIN-CCS)
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Extended SigmaCCS architecture (adds fingerprints, descriptors, instrument
    type) benchmarked in the same cross-database aggregation (expected R2 ≈ 0.84 on
    CCSBase→METLIN-CCS)
- name: Python
  role: Runs the scripts/train-test.py train/evaluate script and the aggregation notebooks
    via poetry
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit (v2023.09.5)
  role: Reads SMILES representations of each molecule during data processing into
    the parquet inputs consumed by the cross-database runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:6ef94fb2ab63fab267a60dc75bcd16da3844496187821c2457c5c3a48172aaae
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-dataset-generalization-metrics-aggregation

## Summary

Aggregates R2, MAE, and MRE metrics from cross-database CCS prediction runs — training GNN models (SigmaCCS, GraphCCS, Mol2CCS) on one CCS database (e.g. CCSBase) and evaluating on a structurally different one (e.g. METLIN-CCS) — to quantify generalization failure matching the paper's Fig. 3.

## When to use

Use when a trained GNN CCS predictor must be evaluated on a structurally different database than its training set (e.g. CCSBase→METLIN-CCS or METLIN-CCS→CCSBase) and per-model R2/MAE/MRE must be compiled into a cross-database comparison table.

## When NOT to use

- When evaluating in-distribution performance (train and test on the same database split, e.g. train-metlin-test-metlin) — the skill targets cross-chemical-space generalization, not standard held-out accuracy
- When the goal is to improve generalization by adding features (fingerprints, descriptors, instrument type in Mol2CCS) rather than to quantify the generalization gap itself
- When the datasets are not structurally distinct CCS databases — aggregating metrics across random splits of one database will not reproduce the reported generalization drop

## Inputs

- Formatted training parquet file for one database (e.g. ccs-prediction/metlin_train_3d.parquet) with smiles, adduct, ccs, and coordinates columns
- Formatted test parquet file for the other database (e.g. ccs-prediction/ccsbase_3d.parquet)
- Trained model file (.h5) and parameter JSON per train/test direction
- Trained models from SigmaCCS, GraphCCS, and Mol2CCS architectures

## Outputs

- Per-model evaluation metrics (R2, MAE, MRE) for each train-database→test-database direction
- Aggregated cross-database metrics table reproducing the Fig. 3 generalization comparison
- Prefix-keyed prediction output files of the test set (e.g. train-metlin-test-ccsbase.out)

## How to apply

Run the training/evaluation cycle via `poetry run python scripts/train-test.py` with a direction-encoding prefix (e.g. 'train-ccsbase-test-metlin' or 'train-metlin-test-ccsbase'), passing the formatted parquet files (e.g. metlin_train_3d.parquet, ccsbase_3d.parquet), the parameter JSON path, the model output .h5 path, and column names for smiles, adduct, ccs, and 3d coordinates, with dropout rate 0.1 and 400 epochs. Evaluate each trained model on the held-out other-database test set and record R2, MAE, and MRE per model. Repeat the cycle with the two databases' roles reversed (e.g. train on METLIN-CCS, test on CCSBase). Aggregate all cross-direction metrics into a single table and validate against the expected Fig. 3 drops (R2 ≈ 0.36 for GraphCCS, ≈ 0.8 for SigmaCCS, ≈ 0.84 for Mol2CCS for CCSBase→METLIN-CCS). The rationale is that models accurate within their training chemical space collapse on structurally novel regions, so both directions must be tested to characterize generalizability.

## Related tools

- **SigmaCCS** (GNN model trained and cross-evaluated; one of the three models whose R2/MAE/MRE are aggregated (expected R2 ≈ 0.8 on CCSBase→METLIN-CCS)) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (GNN model trained and cross-evaluated; showed the steepest generalization drop (expected R2 ≈ 0.36 on CCSBase→METLIN-CCS)) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Extended SigmaCCS architecture (adds fingerprints, descriptors, instrument type) benchmarked in the same cross-database aggregation (expected R2 ≈ 0.84 on CCSBase→METLIN-CCS))
- **Python** (Runs the scripts/train-test.py train/evaluate script and the aggregation notebooks via poetry) — https://github.com/enveda/ccs-prediction
- **RDKit (v2023.09.5)** (Reads SMILES representations of each molecule during data processing into the parquet inputs consumed by the cross-database runs)

## Examples

```
make train-metlin-test-metlin  # or: poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- The aggregated table reproduces the CCSBase→METLIN-CCS direction of Fig. 3: R2 ≈ 0.36 for GraphCCS, ≈ 0.8 for SigmaCCS, ≈ 0.84 for Mol2CCS
- Both directions are present in the table (CCSBase→METLIN-CCS and METLIN-CCS→CCSBase), since the paper observed similar lack of generalizability in the reverse direction
- Each model row reports all three metrics: R2, MAE, and MRE
- All cross-database R2 values are substantially lower than the in-database (train-metlin-test-metlin style) baselines, confirming the generalization drop
- Prediction output files exist for every prefix-keyed run (e.g. train-metlin-test-ccsbase.out)

## Limitations

- The dramatic performance drops reflect lack of generalizability of models trained on CCSBase to other chemical spaces, not a defect of the aggregation procedure itself
- METLIN-CCS focuses on synthetic small molecules, so the evaluation is restricted to small molecules and specific regions of chemical space
- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS values used to train the models, inflating cross-database error

## Evidence

- [discussion] the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
- [discussion] We observed similar results when training on METLIN-CCS and evaluating on CCSBase: "We observed similar results when training on METLIN-CCS and evaluating on CCSBase"
- [abstract] their performance significantly declines when applied to structurally novel regions: "our results confirm the high accuracy of these models within chemical spaces similar to their training environments, their performance significantly declines when applied to structurally novel regions"
- [readme] See the commands in the `Makefile` to train the models. Run them as `make train-metlin-test-metlin`: "See the commands in the `Makefile` to train the models. Run them as `make train-metlin-test-metlin`"
- [methods] Run the notebooks located in the `notebooks` corresponding to each analysis.: "Run the notebooks located in the `notebooks` corresponding to each analysis."
