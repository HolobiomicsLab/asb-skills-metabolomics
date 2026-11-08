---
name: cross-database-record-count-verification
description: Use when reproducing a data-processing pipeline from a manuscript and
  you need to confirm the cleaned dataset size matches the reported figure (e.g.,
  CCSBase 16,989 → 13,617 points) before any model training.
when_to_use_negative:
- The database version differs from the one cited in the paper (e.g., a newer CCSBase
  release) — the exact count invariant 16,989 → 13,617 will not hold and should not
  be treated as an error signal
- You are evaluating model prediction accuracy or generalizability rather than verifying
  data preprocessing provenance
- The dataset is already standardized (e.g., using the Zenodo-published predictions/splits
  at 10.5281/zenodo.11199061) — re-counting would be redundant
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: pandas
  role: Loads the raw excel/csv database dumps and tabulates record counts at each
    preprocessing step
- name: RDKit (v2023.09.5)
  role: Parses each SMILES to detect and drop entries with missing or disconnected
    molecular graphs
  repo: https://rdkit.org/
- name: Mol2CCS
  role: Defines the target input schema (smiles, adduct, ccs, optional 3D coordinates)
    that the verified record counts must populate as parquet files
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:a46948a0b86c31941ef5953fb99df9121a3649f6ac4010d35e085846a87d25a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mol2CCS
  - Python
  - RDKit (v2023.09.5)
  - pandas
  - poetry
  - pre-commit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Cross-database record count verification

## Summary

Verifies that a database preprocessing/standardization chain (deduplication, filtering, aggregation) reproduces the exact record counts reported in a paper — here, confirming CCSBase v1.3 shrinks from 16,989 raw CCS values to 13,617 cleaned data points before model training.

## When to use

Use when reproducing a data-processing pipeline from a manuscript and you need to confirm the cleaned dataset size matches the reported figure (e.g., CCSBase 16,989 → 13,617 points) before any model training.

## When NOT to use

- The database version differs from the one cited in the paper (e.g., a newer CCSBase release) — the exact count invariant 16,989 → 13,617 will not hold and should not be treated as an error signal
- You are evaluating model prediction accuracy or generalizability rather than verifying data preprocessing provenance
- The dataset is already standardized (e.g., using the Zenodo-published predictions/splits at 10.5281/zenodo.11199061) — re-counting would be redundant

## Inputs

- Raw CCSBase v1.3 dump (excel/csv) downloaded from ccsbase.net — 16,989 CCS values, 22 datasets, 6,744 distinct molecules, three instruments
- Raw METLIN-CCS dump (excel/csv) downloaded from metlin.scripps.edu
- Paper-reported record counts to verify against (e.g., 13,617 cleaned CCSBase data points)

## Outputs

- Cleaned CCSBase dataset with 13,617 data points (missing/disconnected SMILES dropped, SMILES–adduct duplicates aggregated by mean CCS, MAD threshold > 5 removed)
- Scaffold-stratified 80/20 split parquet files (ccsbase_train.parquet, ccsbase_test.parquet, metlin_train_3d.parquet, metlin_test_3d.parquet) in Mol2CCS schema
- Record-count log at each filtering step for provenance

## How to apply

Download the raw database dump (CCSBase from ccsbase.net or METLIN-CCS from metlin.scripps.edu) as excel/csv and load it with pandas in the paper's data_processing notebooks. Apply each documented filtering step in order, counting records after each: read each SMILES with RDKit (v2023.09.5) and drop entries with missing or disconnected molecular graphs, then deduplicate repeated SMILES–adduct pairs by computing the mean CCS value per pair (the paper's pipeline reduces CCSBase from 16,989 to 13,617). Compare the intermediate and final counts against the published numbers (16,989 raw values across 22 datasets, 6,744 distinct molecules, three instruments → 13,617 cleaned points). Finally, confirm the cleaned records format into the Mol2CCS input schema (smiles, adduct, ccs, optional 3D coordinates) as the expected parquet outputs, e.g. ccsbase_train.parquet/ccsbase_test.parquet and metlin_train_3d.parquet/metlin_test_3d.parquet.

## Related tools

- **pandas** (Loads the raw excel/csv database dumps and tabulates record counts at each preprocessing step)
- **RDKit (v2023.09.5)** (Parses each SMILES to detect and drop entries with missing or disconnected molecular graphs) — https://rdkit.org/
- **Mol2CCS** (Defines the target input schema (smiles, adduct, ccs, optional 3D coordinates) that the verified record counts must populate as parquet files) — https://github.com/enveda/ccs-prediction

## Examples

```
jupyter nbconvert --to notebook --execute notebooks/data_processing/2_data_splits.ipynb
```

## Evaluation signals

- Raw CCSBase count equals 16,989 CCS values across 22 datasets, 6,744 distinct molecules, and three instruments (v1.3)
- After dropping missing/disconnected SMILES and deduplicating SMILES–adduct pairs (removing duplicates with mean absolute deviation > 5, assigning mean CCS to the rest), the count is exactly 13,617
- Output parquet files follow the Mol2CCS schema with columns smiles, adduct, ccs (and coordinates when --coordinates-present is used)
- The notebooks in notebooks/data_processing execute end-to-end and regenerate the manuscript's data splits
- Row counts after each filtering step are logged and sum/decrease consistently from 16,989 down to 13,617 with no silent drops

## Limitations

- Counts are version-locked: the 16,989 → 13,617 invariant holds only for CCSBase v1.3 and the paper's exact MAD > 5 deduplication rule
- Variations in experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values, so count verification alone does not guarantee data quality
- The scope is restricted mainly to small molecules and specific regions of chemical space (e.g., METLIN-CCS is based on synthetic structures), so verified counts say nothing about coverage of structurally novel regions

## Evidence

- [intro] CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments from 6,744 distinct molecules: "CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments from 6,744 distinct molecules"
- [methods] Each notebook reads the csv/excel and formats it according to the input of Mol2CCS.: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [methods] Each user should download the raw database (as excel/csv) and read them in the two notebooks for each database: "Each user should download the raw database (as excel/csv) and read them in the two notebooks for each database"
- [readme] data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits.: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
