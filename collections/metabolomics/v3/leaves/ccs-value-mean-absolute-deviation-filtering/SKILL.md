---
name: ccs-value-mean-absolute-deviation-filtering
description: Use when a CCS training table (e.g. CCSBase parquet/CSV with 'ccs', 'smiles',
  'adduct' columns) contains multiple experimental records per SMILES-adduct pair
  and inconsistent replicate values must be removed before model training.
when_to_use_negative:
- Input is already deduplicated or a split training/test file (e.g. metlin_train_3d.parquet,
  ccsbase_3d.parquet used by train-test.py) — no duplicate collapsing is needed
- Applying the threshold-5 dedup rule to METLIN-CCS or other non-CCSBase datasets
  without first re-analyzing the deviation distribution, since the 5 cutoff was derived
  from CCSBase duplicate histograms
- The goal is filtering model predictions by confidence rather than cleaning experimental
  training records — use the confidence-model filtering approach instead
edam_operation: http://edamontology.org/operation_3661
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3318
tools:
- name: Python
  role: Implements the pandas group-by/deduplication and threshold filtering on the
    CCSBase table
- name: poetry
  role: Development environment used to run the data-processing scripts/notebooks
  repo: https://github.com/enveda/ccs-prediction
- name: Mol2CCS
  role: Downstream model whose training input format the deduplicated CCSBase table
    is formatted for by the data_processing notebooks
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit (v2023.09.5)
  role: Reads SMILES representations of each molecule when processing the database
    tables
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:959d860a62b63fc761e707f49ddb2162e7518e98eec33569bc01e4449297fb97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - poetry
  edam_operation: http://edamontology.org/operation_3661
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ccs-value-mean-absolute-deviation-filtering

## Summary

Data-standardization skill for removing inconsistent duplicate SMILES-adduct CCS records from CCSBase (v1.3): duplicate pairs with a mean absolute CCS deviation greater than 5 are dropped, and remaining duplicates are collapsed to a single record with the mean CCS value before GNN training.

## When to use

Use when a CCS training table (e.g. CCSBase parquet/CSV with 'ccs', 'smiles', 'adduct' columns) contains multiple experimental records per SMILES-adduct pair and inconsistent replicate values must be removed before model training.

## When NOT to use

- Input is already deduplicated or a split training/test file (e.g. metlin_train_3d.parquet, ccsbase_3d.parquet used by train-test.py) — no duplicate collapsing is needed
- Applying the threshold-5 dedup rule to METLIN-CCS or other non-CCSBase datasets without first re-analyzing the deviation distribution, since the 5 cutoff was derived from CCSBase duplicate histograms
- The goal is filtering model predictions by confidence rather than cleaning experimental training records — use the confidence-model filtering approach instead

## Inputs

- Processed CCSBase (v1.3) table as parquet or CSV/Excel containing 'ccs', 'smiles', and 'adduct' columns

## Outputs

- ccsbase_deduplicated.parquet — deduplicated table with 'ccs', 'smiles', 'adduct' columns where all retained duplicate SMILES-adduct pairs have mean absolute CCS deviation <= 5

## How to apply

Load the processed CCSBase table into pandas and group entries by SMILES-adduct pair, keeping only groups with more than one record. For each duplicate group, compute the mean absolute deviation of the CCS values; the threshold of 5 was chosen through histogram analysis of the deviations across all duplicates. Drop duplicate pairs whose mean absolute CCS deviation exceeds 5 — these reflect conflicting measurements, e.g. from variations in instrument types. For retained duplicates, assign the mean CCS value to a single deduplicated record. Save the result with the 'ccs', 'smiles', and 'adduct' columns as 'ccsbase_deduplicated.parquet'. This mirrors the filtering strategies the authors applied to mitigate inaccuracies in experimental CCS training data.

## Related tools

- **Python** (Implements the pandas group-by/deduplication and threshold filtering on the CCSBase table)
- **poetry** (Development environment used to run the data-processing scripts/notebooks) — https://github.com/enveda/ccs-prediction
- **Mol2CCS** (Downstream model whose training input format the deduplicated CCSBase table is formatted for by the data_processing notebooks) — https://github.com/enveda/ccs-prediction
- **RDKit (v2023.09.5)** (Reads SMILES representations of each molecule when processing the database tables) — https://github.com/enveda/ccs-prediction

## Examples

```
dup = df.groupby(['smiles','adduct'])['ccs'].agg(lambda v: v.diff().abs().mean() if len(v)>1 else 0); df[~(df.set_index(['smiles','adduct']).index.isin(dup[dup>5].index) & df.duplicated(['smiles','adduct'], keep=False))]
```

## Evaluation signals

- Output file 'ccsbase_deduplicated.parquet' exists and is a valid parquet/CSV containing exactly the 'ccs', 'smiles', and 'adduct' columns
- All retained duplicate SMILES-adduct pairs in the output have mean absolute CCS deviation <= 5
- No SMILES-adduct group in the output contains more than one record (except where duplicates were collapsed to a mean-value record)
- Records removed are exactly those duplicate groups whose deviation histogram justified the threshold of 5
- Downstream data-processing notebooks can read the deduplicated table and format it to the Mol2CCS input schema

## Limitations

- The threshold of 5 was determined by histogram analysis of CCSBase duplicate deviations; it is dataset-specific and may not transfer to other CCS databases such as METLIN-CCS
- Variations in experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values, so filtering by deviation mitigates but does not eliminate measurement inconsistency
- Retained duplicate pairs are collapsed by mean CCS value, which discards instrument-type provenance unless additional columns are preserved

## Evidence

- [discussion] filtering of duplicate CCSBase pairs: "We applied filtering strategies similar to those used in previous studies (see Section."Standardization across datasets") to try and mitigate this issue"
- [discussion] rationale: instrument-driven inaccuracies in CCS values: "variations in the experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values used to train our models"
- [methods] data_processing notebooks process CCSBase and format for Mol2CCS: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [methods] notebook location for database processing: "located at https://github.com/enveda/ccs-prediction/tree/main/notebooks/data_processing"
