---
name: smiles-adduct-deduplication-with-mean-ccs-averaging
description: Use when a raw CCS database export (e.g., ccsbase.net or metlin.scripps.edu
  excel/csv dumps) contains repeated SMILES-adduct pairs or unparseable SMILES, and
  you need a single cleaned value per pair before model training or splitting.
when_to_use_negative:
- The input already contains one row per unique SMILES-adduct pair with a single CCS
  value — deduplication would be a no-op and averaging could silently alter curated
  values
- You need to preserve individual replicate CCS measurements (e.g., to study inter-instrument
  variability across the three CCSBase instruments) rather than collapse them to means
- The target model schema requires raw multi-instance training data rather than Mol2CCS's
  smiles/adduct/ccs/coordinates parquet format
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: RDKit (v2023.09.5)
  role: Reads each SMILES and drops entries with missing or disconnected molecular
    graphs before deduplication
  repo: https://github.com/enveda/ccs-prediction
- name: Mol2CCS
  role: Defines the target input schema (smiles, adduct, ccs, optional 3D coordinates)
    that the cleaned data are formatted into
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Runs the data_processing notebooks that load the raw dumps and perform the
    filtering/deduplication
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:08d613b06ff76cbb48549362b2433bbc3a26bbf3a18c4a5951b7ca60888c9cb3
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# SMILES-adduct deduplication with mean CCS averaging

## Summary

A data-standardization step that cleans raw CCS database dumps (CCSBase, METLIN-CCS) by dropping rows with missing or disconnected SMILES and collapsing duplicate SMILES-adduct pairs into a single record assigned the mean of the measured CCS values. Applying it to CCSBase v1.3 reduces 16,989 raw CCS values to 13,617 cleaned data points.

## When to use

Use when a raw CCS database export (e.g., ccsbase.net or metlin.scripps.edu excel/csv dumps) contains repeated SMILES-adduct pairs or unparseable SMILES, and you need a single cleaned value per pair before model training or splitting.

## When NOT to use

- The input already contains one row per unique SMILES-adduct pair with a single CCS value — deduplication would be a no-op and averaging could silently alter curated values
- You need to preserve individual replicate CCS measurements (e.g., to study inter-instrument variability across the three CCSBase instruments) rather than collapse them to means
- The target model schema requires raw multi-instance training data rather than Mol2CCS's smiles/adduct/ccs/coordinates parquet format

## Inputs

- Raw CCSBase v1.3 dump (excel/csv from https://ccsbase.net/query; 16,989 CCS values, 22 datasets, 6,744 distinct molecules, three instruments)
- Raw METLIN-CCS dump (excel/csv from https://metlin.scripps.edu/)

## Outputs

- Cleaned per-database dataframe of unique SMILES-adduct pairs with mean-assigned CCS values (CCSBase: 13,617 rows)
- Parquet files in the Mol2CCS input schema (e.g., ccsbase_train.parquet / ccsbase_test.parquet, metlin_train_3d.parquet / metlin_test_3d.parquet)

## How to apply

Load the raw database dump (excel/csv) with pandas, then read each SMILES with RDKit (v2023.09.5), dropping entries with missing or disconnected molecular graphs. For repeated SMILES-adduct pairs, compute the mean absolute deviation across measurements and remove pairs with deviation greater than five; for remaining duplicates, assign the mean CCS value per pair. This mirrors the paper's 'Standardization across datasets' filtering used to mitigate instrument-driven inaccuracies in CCS training values. Output the cleaned records into the Mol2CCS input schema (smiles, adduct, ccs, optional 3D coordinates) as parquet files before computing Murcko-scaffold-stratified 80/20 splits.

## Related tools

- **RDKit (v2023.09.5)** (Reads each SMILES and drops entries with missing or disconnected molecular graphs before deduplication) — https://github.com/enveda/ccs-prediction
- **Mol2CCS** (Defines the target input schema (smiles, adduct, ccs, optional 3D coordinates) that the cleaned data are formatted into) — https://github.com/enveda/ccs-prediction
- **Python** (Runs the data_processing notebooks that load the raw dumps and perform the filtering/deduplication) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python -m notebooks/data_processing/1_data_processing  # load ccsbase.net excel/csv with pandas, filter unparseable SMILES with RDKit, deduplicate SMILES-adduct pairs by mean CCS
```

## Evaluation signals

- Record-count invariant: CCSBase rows must drop from 16,989 raw values to exactly 13,617 cleaned data points
- No duplicate SMILES-adduct pairs remain in the output; each surviving pair carries a single mean CCS value
- Rows with missing or disconnected SMILES (unparseable by RDKit v2023.09.5) are all removed before counting
- Deduplication rule check: duplicate SMILES-adduct pairs with mean absolute deviation greater than five were removed, not averaged
- Output parquet files match the Mol2CCS schema (smiles, adduct, ccs, optional coordinates column) consumed by 2_data_splits.ipynb

## Limitations

- Averaging duplicates discards measurement variance and inter-instrument differences; the paper notes variations in experimental conditions such as instrument types likely introduced inaccuracies in the CCS training values
- The filtering thresholds (deviation > 5 removal, mean assignment) were designed for CCSBase/METLIN-CCS-style CCS data and may not transfer to other measured-property databases
- The cleaned dataset scope is small molecules in specific regions of chemical space (METLIN-CCS is based on synthetic structures), limiting downstream generalizability

## Evidence

- [intro] we first read the SMILES representation of each molecule using RDKit (v2023.09.5): "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
- [methods] Each notebook reads the csv/excel and formats it according to the input of Mol2CCS.: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [intro] CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments from 6,744 distinct molecules: "CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments"
- [discussion] We applied filtering strategies similar to those used in previous studies (see Section."Standardization across datasets") to try and mitigate this issue: "We applied filtering strategies similar to those used in previous studies (see Section."Standardization across datasets")"
- [discussion] variations in the experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values: "variations in the experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values"
- [readme] see notebooks/data_processing/2_data_splits.ipynb for details on the format: "see notebooks/data_processing/2_data_splits.ipynb for details on the format"
