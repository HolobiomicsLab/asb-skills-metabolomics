---
name: cross-dataset-generalization-evaluation
description: Use when you need to test whether a CCS prediction model's reported accuracy
  transfers beyond its training dataset's chemical space, using METLIN-CCS (~27k structures)
  and CCSBase v1.3 (16,989 values) as train/test pairs.
when_to_use_negative:
- When the goal is in-distribution accuracy on a single homogeneous dataset rather
  than quantifying the generalization gap across chemical spaces.
- When the molecule classes of interest fall outside small-molecule synthetic chemistry,
  since METLIN-CCS is based on synthetic structures and the evaluation is restricted
  to specific regions of chemical space.
- When experimental CCS references for the target chemical space are unavailable,
  as only a few thousand high-quality experimental values exist and lack of data availability
  is a major field challenge.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Mol2CCS
  role: Extension of SigmaCCS used in cross-dataset regimes, incorporating instrument
    type, molecular fingerprints, descriptors and molecule types to partially improve
    generalizability
  repo: https://github.com/enveda/ccs-prediction
- name: SigmaCCS
  role: Baseline GNN benchmarked in train-on-CCSBase and train-on-METLIN regimes to
    test generalization
  repo: https://github.com/zmzhang/SigmaCCS
- name: GraphCCS
  role: Second GNN benchmarked alongside SigmaCCS and Mol2CCS on METLIN-CCS and CCSBase
  repo: https://github.com/tingxiecsu/GraphCCS
- name: Python
  role: Runtime for scripts/train-test.py training and notebook-based analysis in
    the enveda/ccs-prediction environment
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit (v2023.09.5)
  role: Parses SMILES representations of each molecule during data preprocessing before
    graph construction
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:d173dec8f276a2ef9eae8e86f09ecac8b8669b2a6128e0ff101e77b00e9d021f
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-dataset-generalization-evaluation

## Summary

Benchmarks CCS prediction models (SigmaCCS, GraphCCS, Mol2CCS) by training on one CCS database (METLIN-CCS or CCSBase v1.3) and testing on the other, to quantify how performance degrades outside the training chemical space.

## When to use

Use when you need to test whether a CCS prediction model's reported accuracy transfers beyond its training dataset's chemical space, using METLIN-CCS (~27k structures) and CCSBase v1.3 (16,989 values) as train/test pairs.

## When NOT to use

- When the goal is in-distribution accuracy on a single homogeneous dataset rather than quantifying the generalization gap across chemical spaces.
- When the molecule classes of interest fall outside small-molecule synthetic chemistry, since METLIN-CCS is based on synthetic structures and the evaluation is restricted to specific regions of chemical space.
- When experimental CCS references for the target chemical space are unavailable, as only a few thousand high-quality experimental values exist and lack of data availability is a major field challenge.

## Inputs

- METLIN-CCS experimental CCS dataset (excel/csv download from metlin.scripps.edu)
- CCSBase v1.3 dataset (excel/csv download from ccsbase.net/query)
- Processed train/test parquet files with smiles, adduct, ccs and 3D coordinates columns (e.g. metlin_train_3d.parquet)
- Parameter JSON path for model configuration

## Outputs

- Trained model .h5 file and parameter JSON per train/test regime
- Prediction outputs on the held-out test set (prefixed, e.g. train-metlin-test-ccsbase)
- MAE and MRE metrics per cross-dataset regime, comparable to Fig. 5 and Supplementary Figs. 9-13

## How to apply

Run the enveda/ccs-prediction repository's data_processing notebooks to format METLIN-CCS and CCSBase and produce train/test splits, then launch training via the Makefile targets (e.g. make train-metlin-test-metlin) or scripts/train-test.py with explicit train/test parquet inputs, adduct and CCS column names, and hyperparameters (dropout 0.1, 400 epochs). Train on one database and evaluate on the other (train-metlin-test-ccsbase and the reverse) to expose the accuracy drop the paper reports: high accuracy on CCSBase does not generalize to METLIN-CCS chemical space and vice versa. Compute MAE and MRE on each held-out set and compare against the in-distribution baseline to quantify the generalization gap. Optionally extend the architecture (Mol2CCS) with additional features such as instrument type and molecular fingerprints/descriptors to partially close the gap.

## Related tools

- **Mol2CCS** (Extension of SigmaCCS used in cross-dataset regimes, incorporating instrument type, molecular fingerprints, descriptors and molecule types to partially improve generalizability) — https://github.com/enveda/ccs-prediction
- **SigmaCCS** (Baseline GNN benchmarked in train-on-CCSBase and train-on-METLIN regimes to test generalization) — https://github.com/zmzhang/SigmaCCS
- **GraphCCS** (Second GNN benchmarked alongside SigmaCCS and Mol2CCS on METLIN-CCS and CCSBase) — https://github.com/tingxiecsu/GraphCCS
- **Python** (Runtime for scripts/train-test.py training and notebook-based analysis in the enveda/ccs-prediction environment) — https://github.com/enveda/ccs-prediction
- **RDKit (v2023.09.5)** (Parses SMILES representations of each molecule during data preprocessing before graph construction) — https://www.rdkit.org/

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --dropout-rate 0.1 --epochs 400
```

## Evaluation signals

- MAE and MRE computed on the cross-dataset test set are substantially higher than the in-distribution baseline (train-metlin-test-metlin or train-ccsbase-test-ccsbase), matching the direction of the paper's generalization-gap findings.
- Per-regime output files appear with the expected prefix (e.g. train-metlin-test-ccsbase.out and matching prediction outputs) as configured by --prefix.
- The train/test parquet files contain the columns named by --smiles-column-name, --adduct-column-name, --ccs-column-name and, when --coordinates-present is set, --coordinates-column-name.
- Reproducibility check: the exact predictions used in the paper are downloadable from Zenodo DOI 10.5281/zenodo.11199061 and can be unzipped into the data directory to compare against re-trained outputs.
- Adding Mol2CCS extensions (instrument type, fingerprints, descriptors, molecule type) partially reduces the cross-dataset error relative to the SigmaCCS baseline.

## Limitations

- The original high accuracy reported when models were trained on CCSBase does not generalize to other chemical spaces, and similar degradation is seen when training on METLIN-CCS and evaluating on CCSBase.
- Evaluation is restricted in scope mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures).
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values; filtering strategies similar to those in 'Standardization across datasets' were applied to mitigate this.
- The METLIN-CCS dataset offers several times more data points than CCSBase, yet lack of generalizability is still observed, so dataset size alone does not guarantee transfer.

## Evidence

- [results] Benchmarking GNN models on two CCS databases: "we benchmarked two GNNs (i.e., SigmaCCS and GraphCCS) and Mol2CCS, an adaptation of SigmaCCS, on METLIN-CCS and CCSBase"
- [discussion] Accuracy does not transfer across chemical spaces: "the original high accuracy reported when the models were trained on CCSBase does not generalize to other chemical spaces"
- [readme] Concrete training invocation from README: "poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet""
- [readme] Data processing and split generation notebooks: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
- [discussion] Scope restriction of the evaluation: "Another limitation of our evaluation is its restriction in scope, as it mainly focuses on small molecules and specific regions of the chemical space (e.g., METLIN-CCS is based on synthetic structures)"
- [discussion] Cross-dataset degradation is symmetric: "We observed similar results when training on METLIN-CCS and evaluating on CCSBase"
