---
name: regression-metrics-computation
description: Use when scoring trained CCS prediction models (SigmaCCS, GraphCCS, Mol2CCS)
  on test-split parquet files and you need R2, RMSE, and MRE tabulated per model and
  per database (CCSBase, METLIN-CCS, or combined).
when_to_use_negative:
- 'When the analysis goal is cross-database generalizability assessment rather than
  within-database accuracy: the paper shows the high within-database scores (R2 slightly
  below 0.986 on CCSBase) do not generalize to other chemical spaces, so same-database
  metrics alone would overstate model quality.'
- When predictions have no paired experimental CCS references (e.g. the SigmaCCS dataset
  of 282 million predicted CCS values for 94 million compounds) — R2, RMSE, and MRE
  all require experimental ground truth.
- When outputs are classification/filtering results (e.g. confidence-model-filtered
  predictions) rather than continuous CCS regression values.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3473
tools:
- name: Python
  role: Language in which the metrics computation from test predictions and assembly
    of the metrics tables are performed, invoked via `poetry run python`
  repo: https://github.com/enveda/ccs-prediction
- name: SigmaCCS
  role: GNN model whose test-split predictions are scored for R2, RMSE, and MRE (e.g.
    MRE of 1.57% when trained on CCSBase)
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: GNN model whose test-split predictions are scored; best model on the combined
    CCSBase + METLIN-CCS dataset
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Mol2CCS
  role: Extension of SigmaCCS whose test-split predictions are scored (R2 = 0.985,
    MRE = 1.51% on CCSBase)
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:ce70bacf84381ec5b48244ce700e632af08aeeea3b9b844987f16d31c74a22a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - CCSbase
  - DeepCCS
  - GraphCCS
  - Mol2CCS
  - Python
  - RDKit
  - SigmaCCS
  - Spektral
  - TensorFlow
  - poetry
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# regression-metrics-computation

## Summary

Computes regression performance metrics (R2, RMSE, and mean relative error, MRE) for the CCS prediction models SigmaCCS, GraphCCS, and Mol2CCS from their test-split predictions, and assembles them into per-database, per-model metrics tables. This is the quantitative endpoint used to judge whether trained and evaluated models reach the accuracy reported within CCSBase, METLIN-CCS, and the combined CCSBase + METLIN-CCS datasets.

## When to use

Use when scoring trained CCS prediction models (SigmaCCS, GraphCCS, Mol2CCS) on test-split parquet files and you need R2, RMSE, and MRE tabulated per model and per database (CCSBase, METLIN-CCS, or combined).

## When NOT to use

- When the analysis goal is cross-database generalizability assessment rather than within-database accuracy: the paper shows the high within-database scores (R2 slightly below 0.986 on CCSBase) do not generalize to other chemical spaces, so same-database metrics alone would overstate model quality.
- When predictions have no paired experimental CCS references (e.g. the SigmaCCS dataset of 282 million predicted CCS values for 94 million compounds) — R2, RMSE, and MRE all require experimental ground truth.
- When outputs are classification/filtering results (e.g. confidence-model-filtered predictions) rather than continuous CCS regression values.

## Inputs

- Test-split parquet files (from notebooks/data_processing/2_data_splits.ipynb) with columns for 3D coordinates, SMILES, adduct, and CCS
- Prediction files for the test set generated per run prefix (e.g. prefix 'train-metlin-test-metlin')
- Trained model files (.h5) and parameter JSON files for the models being scored

## Outputs

- metrics_table.csv (per-database, per-model R2 and MRE)
- metrics_table.tsv (one row per model with columns R2, RMSE, and MRE for combined-dataset runs)

## How to apply

After models are trained via the Makefile commands or `scripts/train-test.py` (dropout rate 0.1, 400 epochs) and predictions are written with per-run prefixes (e.g. `train-metlin-test-metlin`), load each test split — the parquet files carrying the `coordinates`, `smiles`, `adduct`, and `ccs` columns — and score each model's predictions against the experimental CCS values. For same-database evaluation compute R2 and mean relative error (MRE) per model per database; for the combined CCSBase + METLIN-CCS benchmark also report RMSE. Assemble the per-model metrics into a single table (`metrics_table.csv` or `metrics_table.tsv`) with one row per model and columns for R2, RMSE, and MRE. The rationale is that these three metrics are exactly the quantities the paper uses to compare models, so reproducing them at the expected values validates the full training/evaluation pipeline.

## Related tools

- **Python** (Language in which the metrics computation from test predictions and assembly of the metrics tables are performed, invoked via `poetry run python`) — https://github.com/enveda/ccs-prediction
- **SigmaCCS** (GNN model whose test-split predictions are scored for R2, RMSE, and MRE (e.g. MRE of 1.57% when trained on CCSBase)) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (GNN model whose test-split predictions are scored; best model on the combined CCSBase + METLIN-CCS dataset) — https://github.com/tingxiecsu/GraphCCS
- **Mol2CCS** (Extension of SigmaCCS whose test-split predictions are scored (R2 = 0.985, MRE = 1.51% on CCSBase)) — https://github.com/enveda/ccs-prediction

## Evaluation signals

- Same-database CCSBase benchmark: GraphCCS and SigmaCCS achieve R2 slightly below 0.986 with MRE of 1.55% and 1.57% respectively; Mol2CCS achieves R2 = 0.985 and MRE = 1.51%.
- METLIN-CCS-trained models should show R2 around 0.9 on their same-database test split.
- Combined CCSBase + METLIN-CCS runs: each model's test-set R2 close to 0.95, RMSE close to 6, and MRE between 1.39% and 1.71%, matching Fig. 4 of the manuscript.
- metrics_table.csv / metrics_table.tsv exists with one row per model and explicit columns for R2, RMSE, and MRE.
- Cross-database runs (e.g. train-metlin-test-ccsbase) yield visibly degraded metrics relative to the within-database numbers, consistent with the paper's generalizability finding.

## Limitations

- High within-database metrics do not transfer: the accuracy reported when models were trained on CCSBase does not generalize to other chemical spaces, and similar lack of generalizability is observed training on METLIN-CCS and evaluating on CCSBase.
- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS training values, which bounds achievable metric quality.
- Evaluation scope is restricted mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures), so metrics may not hold elsewhere.

## Evidence

- [readme] run prefix workflow: "poetry run python scripts/train-test.py \
	--prefix "train-metlin-test-ccsbase" \
	--train-input-file "ccs-prediction/metlin_train_3d.parquet""
- [discussion] model benchmarking scope: "we benchmarked two GNNs (i.e., SigmaCCS and GraphCCS) and Mol2CCS, an adaptation of SigmaCCS, on METLIN-CCS and CCSBase"
- [abstract] metric threshold sanity check: "our results confirm the high accuracy of these models within chemical spaces similar to their training environments, their performance significantly declines"
