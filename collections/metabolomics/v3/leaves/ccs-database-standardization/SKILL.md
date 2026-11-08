---
name: ccs-database-standardization
description: Use when a raw CCS database download (excel/csv from ccsbase.net or METLIN)
  must be cleaned into a model-ready table and contains duplicate SMILES-adduct entries
  with disagreeing CCS values before GNN training.
when_to_use_negative:
- The input is already a deduplicated, standardized feature/model-input table (e.g.,
  ccsbase_3d.parquet ready for scripts/train-test.py)
- 'The goal is filtering model predictions by confidence (a separate skill: confidence-model-based
  prediction filtering), not cleaning experimental training data'
- Duplicate records are needed as-is because the analysis evaluates measurement reproducibility
  across instruments rather than building a training set
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0154
tools:
- name: Python
  role: Implements pandas grouping by SMILES-adduct pair, deviation computation, and
    deduplication in the data_processing notebooks
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit (v2023.09.5)
  role: Reads the SMILES representation of each molecule during database formatting
    and standardization
- name: Mol2CCS
  role: Downstream consumer whose required input format (csv/excel reformatted with
    'ccs', 'smiles', 'adduct' columns) defines the standardization output
  repo: https://github.com/enveda/ccs-prediction
- name: poetry
  role: Manages the development environment in which the processing notebooks are
    run (poetry install; poetry run pre-commit install)
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:d01b4321ce921062b5fe0ac06073e903a55d0c6d46896483ef77f9a31e367db3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - poetry
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# ccs-database-standardization

## Summary

Standardizes raw CCS databases (e.g., CCSBase v1.3, METLIN-CCS) into a unified SMILES/adduct/CCS schema and removes inconsistent duplicate SMILES-adduct pairs by dropping those whose mean absolute CCS deviation exceeds a threshold of 5, chosen via histogram analysis of duplicate deviations.

## When to use

Use when a raw CCS database download (excel/csv from ccsbase.net or METLIN) must be cleaned into a model-ready table and contains duplicate SMILES-adduct entries with disagreeing CCS values before GNN training.

## When NOT to use

- The input is already a deduplicated, standardized feature/model-input table (e.g., ccsbase_3d.parquet ready for scripts/train-test.py)
- The goal is filtering model predictions by confidence (a separate skill: confidence-model-based prediction filtering), not cleaning experimental training data
- Duplicate records are needed as-is because the analysis evaluates measurement reproducibility across instruments rather than building a training set

## Inputs

- Raw CCSBase v1.3 download (excel/csv from https://ccsbase.net/query)
- Raw METLIN-CCS download (excel/csv from https://metlin.scripps.edu/)
- Processed CCSBase table (parquet/CSV) with SMILES, adduct, and CCS columns

## Outputs

- ccsbase_deduplicated.parquet with 'ccs', 'smiles', and 'adduct' columns
- Database tables formatted to Mol2CCS input specification (per-database processing notebooks)

## How to apply

Load the processed CCSBase table (parquet/CSV) into pandas and group entries by SMILES-adduct pair, identifying groups with more than one record. For each duplicate pair, compute the mean absolute deviation of the CCS values; the threshold of 5 was determined through histogram analysis of the deviations across all duplicates. Drop duplicate pairs whose mean absolute deviation exceeds 5; for retained duplicates, assign the mean CCS value to a single deduplicated record. Read SMILES representations with RDKit (v2023.09.5) and format each database so its columns match Mol2CCS inputs ('ccs', 'smiles', 'adduct'). Save the deduplicated table as 'ccsbase_deduplicated.parquet' as part of the data_processing notebooks that also make train/test splits.

## Related tools

- **Python** (Implements pandas grouping by SMILES-adduct pair, deviation computation, and deduplication in the data_processing notebooks) — https://github.com/enveda/ccs-prediction
- **RDKit (v2023.09.5)** (Reads the SMILES representation of each molecule during database formatting and standardization)
- **Mol2CCS** (Downstream consumer whose required input format (csv/excel reformatted with 'ccs', 'smiles', 'adduct' columns) defines the standardization output) — https://github.com/enveda/ccs-prediction
- **poetry** (Manages the development environment in which the processing notebooks are run (poetry install; poetry run pre-commit install)) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run jupyter execute notebooks/data_processing/1_process_databases.ipynb  # outputs ccsbase_deduplicated.parquet
```

## Evaluation signals

- Output file 'ccsbase_deduplicated.parquet' exists and is a valid parquet/CSV with exactly the expected 'ccs', 'smiles', 'adduct' columns
- All retained duplicate SMILES-adduct pairs have mean absolute CCS deviation ≤ 5
- No SMILES-adduct pair appears more than once in the output table
- Retained duplicate values equal the mean of their original CCS values
- The standardized table loads successfully into the Mol2CCS data-processing pipeline (notebooks/data_processing/2_data_splits.ipynb)

## Limitations

- Instrument-type variation in experimental conditions (e.g., different IM-MS instruments in CCSBase v1.3's three instruments) likely introduced inaccuracies in the CCS values; the deviation threshold removes gross conflicts but cannot correct systematic instrument-dependent offsets
- The threshold of 5 was calibrated by histogram analysis of deviations across all CCSBase duplicates; transferring it to other databases (e.g., METLIN-CCS) may require recalibration
- Filtering strategies similar to those used in previous studies were applied to mitigate inaccuracy, but residual noise from inconsistent experimental conditions may remain in the training data

## Evidence

- [other] Duplicate SMILES-adduct pairs in CCSBase were filtered by removing any pairs whose mean absolute deviation of CCS values was greater than five, a threshold determined through histogram analysis: [evidence span withheld — anchor status: not_found]
- [methods] Each notebook reads the csv/excel and formats it according to the input of Mol2CCS.: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [discussion] variations in the experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values used to train our models: "variations in the experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values used to train our models"
- [methods] data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits.: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
