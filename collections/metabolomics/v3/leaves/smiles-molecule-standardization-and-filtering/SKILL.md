---
name: smiles-molecule-standardization-and-filtering
description: Use when raw database exports (e.g., CCSBase from ccsbase.net or METLIN-CCS
  from metlin.scripps.edu as excel/csv) contain invalid or duplicated SMILES–adduct
  entries that must be cleaned before splitting into model train/test sets.
when_to_use_negative:
- Input is already a cleaned, deduplicated feature/split table in Mol2CCS schema —
  re-running the filter would remove legitimate repeated measurements
- Data contains only predicted CCS values (e.g., SigmaCCS's 282-million-value dataset)
  rather than experimental measurements where MAD-based duplicate filtering is meaningful
- SMILES entries are intentionally multi-component (e.g., salts/mixtures you wish
  to keep) since this chain drops disconnected molecular graphs
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0081
- http://edamontology.org/topic_0154
tools:
- name: RDKit (v2023.09.5)
  role: Reads each SMILES and detects missing/disconnected molecular graphs to drop
  repo: https://github.com/enveda/ccs-prediction
- name: Mol2CCS
  role: Defines the target input schema (smiles, adduct, ccs, 3D coordinates) the
    cleaned data is formatted into
  repo: https://github.com/enveda/ccs-prediction
- name: Python
  role: Runs the data_processing notebooks (pandas + RDKit) via poetry
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:f6ae653cbdd71c7595a03fea315d606a8b2c6508a2c820b423863780635695c3
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0081
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# SMILES molecule standardization and filtering

## Summary

A preprocessing chain that standardizes raw CCS database dumps (CCSBase v1.3, METLIN-CCS) by validating SMILES with RDKit v2023.09.5, removing missing/disconnected molecular graphs, and collapsing duplicate SMILES–adduct pairs (removing those with mean absolute deviation > 5) to a mean CCS value.

## When to use

Use when raw database exports (e.g., CCSBase from ccsbase.net or METLIN-CCS from metlin.scripps.edu as excel/csv) contain invalid or duplicated SMILES–adduct entries that must be cleaned before splitting into model train/test sets.

## When NOT to use

- Input is already a cleaned, deduplicated feature/split table in Mol2CCS schema — re-running the filter would remove legitimate repeated measurements
- Data contains only predicted CCS values (e.g., SigmaCCS's 282-million-value dataset) rather than experimental measurements where MAD-based duplicate filtering is meaningful
- SMILES entries are intentionally multi-component (e.g., salts/mixtures you wish to keep) since this chain drops disconnected molecular graphs

## Inputs

- Raw CCSBase (v1.3) dump as excel/csv from https://ccsbase.net/query
- Raw METLIN-CCS dump as excel/csv from https://metlin.scripps.edu/
- Columns: SMILES string, adduct type, experimental CCS value

## Outputs

- Cleaned dataframe of unique SMILES–adduct pairs with mean CCS values (CCSBase: 13,617 rows)
- Parquet files in Mol2CCS input schema (e.g., ccsbase_train.parquet / ccsbase_test.parquet, metlin_train_3d.parquet / metlin_test_3d.parquet)

## How to apply

Load the raw excel/csv dump with pandas, then read each SMILES with RDKit (v2023.09.5), dropping rows where the SMILES is missing or yields a disconnected molecular graph. For repeated SMILES–adduct pairs, compute the mean absolute deviation of their CCS measurements and discard pairs with MAD greater than five; for remaining duplicates, assign the mean CCS value. On CCSBase v1.3 this chain reduces 16,989 raw values (22 datasets, 6,744 distinct molecules, three instruments) to 13,617 cleaned data points. Format the cleaned table into the Mol2CCS input schema (smiles, adduct, ccs, optional 3D coordinates) before scaffold-stratified splitting.

## Related tools

- **RDKit (v2023.09.5)** (Reads each SMILES and detects missing/disconnected molecular graphs to drop) — https://github.com/enveda/ccs-prediction
- **Mol2CCS** (Defines the target input schema (smiles, adduct, ccs, 3D coordinates) the cleaned data is formatted into) — https://github.com/enveda/ccs-prediction
- **Python** (Runs the data_processing notebooks (pandas + RDKit) via poetry) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python -c "import pandas as pd; from rdkit import Chem; df = pd.read_excel('ccsbase_raw.xlsx'); df['mol'] = df.smiles.apply(Chem.MolFromSmiles); df = df[df.mol.apply(lambda m: m is not None)]"
```

## Evaluation signals

- CCSBase record count drops from 16,989 raw values to 13,617 cleaned data points
- No rows remain with missing or disconnected SMILES (RDKit MolFromSmiles returns None or a disconnected graph)
- No duplicate SMILES–adduct pairs remain; each surviving pair carries its mean CCS value
- Removed duplicate pairs had mean absolute deviation greater than five
- Notebooks in notebooks/data_processing run end-to-end and produce the manuscript's parquet splits

## Limitations

- Variations in experimental conditions, such as instrument types, likely introduced inaccuracies in the CCS values, so the MAD > 5 threshold does not remove all measurement noise
- Standardization applied here is specific to SMILES graph validity and duplicate collapsing; no charge/salt/tautomer normalization is described
- Dropping duplicate SMILES–adduct pairs discards information about measurement variance

## Evidence

- [intro] we first read the SMILES representation of each molecule using RDKit (v2023.09.5): "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
- [methods] Each user should download the raw database (as excel/csv) and read them in the two notebooks for each database: "Each user should download the raw database (as excel/csv) and read them in the two notebooks for each database"
- [intro] CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments from 6,744 distinct molecules: "CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments from 6,744 distinct molecules"
- [methods] Each notebook reads the csv/excel and formats it according to the input of Mol2CCS.: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [methods] data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits.: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
