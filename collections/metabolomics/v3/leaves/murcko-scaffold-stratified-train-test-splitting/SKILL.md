---
name: murcko-scaffold-stratified-train-test-splitting
description: Use when preparing CCSBase (16,989 raw CCS values, 22 datasets, 6,744
  distinct molecules) or METLIN-CCS dumps for model benchmarking and you need train/test
  splits that keep scaffold space shared across both partitions.
when_to_use_negative:
- You need an external generalization benchmark (e.g. train on METLIN-CCS and evaluate
  on CCSBase) rather than an in-distribution split — the paper's key finding is that
  scaffold-sharing splits overstate performance on structurally novel regions.
- Your data has not yet passed the preprocessing chain (missing/disconnected SMILES
  removal, SMILES–adduct deduplication with mean absolute deviation threshold of five);
  scaffold-splitting raw duplicated records will leak near-identical measurements
  across splits.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3332
tools:
- name: RDKit (v2023.09.5)
  role: Parses each SMILES to validate the molecular graph (dropping missing/disconnected
    entries) and computes Murcko scaffolds used for stratification
- name: Mol2CCS
  role: Defines the target input schema (smiles, adduct, ccs, optional 3D coordinates)
    that the split parquet files must conform to
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Runs the data_processing notebooks that deduplicate, scaffold-annotate, split,
    and serialize the datasets
  repo: https://github.com/enveda/ccs-prediction
- name: poetry
  role: Installs the reproducible development environment before running the splitting
    notebooks
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:4fb9f34de0fcbf31e14543a8ab5b1a2adf8ccf04ea9053de36f3cc402f8d7fe6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - poetry
  - pre-commit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# murcko-scaffold-stratified-train-test-splitting

## Summary

Splits cleaned CCSBase and METLIN-CCS CCS datasets into 80/20 train/test sets stratified by Murcko scaffold within each molecule-type group, so that both splits share the same scaffold space when benchmarking GNN CCS predictors like SigmaCCS, GraphCCS and Mol2CCS.

## When to use

Use when preparing CCSBase (16,989 raw CCS values, 22 datasets, 6,744 distinct molecules) or METLIN-CCS dumps for model benchmarking and you need train/test splits that keep scaffold space shared across both partitions.

## When NOT to use

- You need an external generalization benchmark (e.g. train on METLIN-CCS and evaluate on CCSBase) rather than an in-distribution split — the paper's key finding is that scaffold-sharing splits overstate performance on structurally novel regions.
- Your data has not yet passed the preprocessing chain (missing/disconnected SMILES removal, SMILES–adduct deduplication with mean absolute deviation threshold of five); scaffold-splitting raw duplicated records will leak near-identical measurements across splits.

## Inputs

- Raw CCSBase dump (ccsbase.net, v1.3; 16,989 CCS values from 22 datasets, 6,744 distinct molecules, three instruments) as excel/csv
- Raw METLIN-CCS dump (metlin.scripps.edu, >27,000 unique structures) as excel/csv
- RDKit-parsable SMILES strings with adduct annotations and experimental CCS values

## Outputs

- ccsbase_train.parquet and ccsbase_test.parquet (80/20 scaffold-stratified split, 13,617 cleaned data points)
- metlin_train_3d.parquet and metlin_test_3d.parquet in the Mol2CCS input schema (smiles, adduct, ccs, optional 3D coordinates)
- Murcko scaffold annotations per molecule used for stratification

## How to apply

First standardize the raw dumps: read SMILES with RDKit (v2023.09.5), drop rows with missing or disconnected molecular graphs, deduplicate repeated SMILES–adduct pairs by computing the mean CCS value per pair (removing pairs with mean absolute deviation greater than five), yielding 13,617 cleaned CCSBase data points from 16,989 raw values. Then compute Murcko scaffolds for each molecule and stratify an 80/20 train/test split within each molecule-type group so scaffold space is shared across the split, avoiding inflated within-scaffold performance estimates. Format the splits into the Mol2CCS input schema — columns for smiles, adduct, ccs, and optional 3D coordinates — as parquet files (e.g. ccsbase_train.parquet / ccsbase_test.parquet and metlin_train_3d.parquet / metlin_test_3d.parquet) using the notebooks in notebooks/data_processing. The splits feed scripts/train-test.py, which consumes them via --train-input-file / --test-input-file arguments for the train-metlin-test-ccsbase style benchmarks.

## Related tools

- **RDKit (v2023.09.5)** (Parses each SMILES to validate the molecular graph (dropping missing/disconnected entries) and computes Murcko scaffolds used for stratification)
- **Mol2CCS** (Defines the target input schema (smiles, adduct, ccs, optional 3D coordinates) that the split parquet files must conform to) — https://github.com/enveda/ccs-prediction
- **Python** (Runs the data_processing notebooks that deduplicate, scaffold-annotate, split, and serialize the datasets) — https://github.com/enveda/ccs-prediction
- **poetry** (Installs the reproducible development environment before running the splitting notebooks) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --ccs-column-name "ccs"
```

## Evaluation signals

- CCSBase record count drops from 16,989 raw values to exactly 13,617 after cleaning (dropping missing/disconnected SMILES, deduplicating SMILES–adduct pairs with mean absolute deviation > 5, assigning mean CCS to remaining duplicates)
- Both train and test parquet files contain the Mol2CCS columns: smiles, adduct, ccs, and (for the *_3d variants) coordinates
- Every SMILES–adduct pair appears at most once per split, and duplicated pairs carry the mean CCS value
- Scaffold space (Murcko scaffolds) is shared across train and test within each molecule-type group
- The notebooks in notebooks/data_processing (including 2_data_splits.ipynb) run end-to-end without errors

## Limitations

- Scaffold-sharing splits keep train and test in a similar structural space, so the high accuracy measured this way does not generalize to structurally novel regions — the paper explicitly shows performance declines outside the training chemical space.
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values used in the splits.
- The evaluation is restricted in scope to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures).

## Evidence

- [methods] Compute Murcko scaffolds and stratify an 80/20 train/test split within each molecule-type group so scaffold space is shared across the split.: [evidence span withheld — anchor status: not_found]
- [results] the number of CCSBase data points was reduced from 16,989 to 13,617: [evidence span withheld — anchor status: not_found]
- [readme] data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits.: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
- [intro] It refers to the concept that both training and validation compounds and their estimated parameters need to be in a similar structural space (scaffold space): "It refers to the concept that both training and validation compounds and their estimated parameters need to be in a similar structural space (scaffold space) and that predicted properties should be"
- [intro] CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments from 6,744 distinct molecules: "CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments from 6,744 distinct molecules"
