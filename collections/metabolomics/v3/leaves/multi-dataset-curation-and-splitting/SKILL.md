---
name: multi-dataset-curation-and-splitting
description: Use when building CCS (or similar molecular property) training corpora
  from multiple public databases (e.g., CCSBase and METLIN-CCS) and you need standardized,
  adduct-aware, split parquet files for model training and benchmarking.
when_to_use_negative:
- You already have harmonized, split parquet files matching the train-test.py schema
  (coordinates, smiles, adduct, ccs) — proceed directly to model training
- Your goal is evaluating cross-dataset generalizability only and you plan to reuse
  the authors' published splits rather than re-curation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3332
tools:
- name: RDKit
  role: Reads SMILES representations and generates the 3D conformations stored in
    the curated parquet files
- name: poetry
  role: Installs the development environment in which the curation and splitting notebooks/scripts
    run
- name: Python
  role: Runs the data_processing notebooks and the downstream training script that
    consumes the splits
  repo: https://github.com/enveda/ccs-prediction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:46bd2b0c8fbb54ed2fe2f85b3dc83127ef3ebb468ffa40ed7986e39b5007b4b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enveda/ccs-prediction
  tools:
  - GraphCCS
  - Python
  - RDKit
  - SigmaCCS
  - poetry
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multi-dataset-curation-and-splitting

## Summary

Curates two heterogeneous CCS databases (CCSBase v1.3 and METLIN-CCS) into harmonized 3D-coordinate parquet files and produces train/test splits that enable combined-dataset training and cross-dataset evaluation of GNN CCS prediction models.

## When to use

Use when building CCS (or similar molecular property) training corpora from multiple public databases (e.g., CCSBase and METLIN-CCS) and you need standardized, adduct-aware, split parquet files for model training and benchmarking.

## When NOT to use

- You already have harmonized, split parquet files matching the train-test.py schema (coordinates, smiles, adduct, ccs) — proceed directly to model training
- Your goal is evaluating cross-dataset generalizability only and you plan to reuse the authors' published splits rather than re-curation

## Inputs

- Raw CCSBase (v1.3) download (excel/csv) from https://ccsbase.net/query
- Raw METLIN-CCS download (excel/csv) from https://metlin.scripps.edu/
- Per-database data_processing notebooks that format records to Mol2CCS input (SMILES, adduct, CCS)
- Optional pre-computed predictions from Zenodo (10.5281/zenodo.11199061) to unzip into the data directory

## Outputs

- Harmonized 3D-coordinate parquet files with coordinates, smiles, adduct, and ccs columns (e.g., metlin_train_3d.parquet, ccsbase_3d.parquet)
- Train/test split parquet files from notebooks/data_processing/2_data_splits.ipynb for combined and cross-dataset configurations

## How to apply

Download the raw CCSBase and METLIN-CCS databases as excel/csv according to their licenses, then read each into its dedicated notebook in notebooks/data_processing to format them to the input schema expected by Mol2CCS/SigmaCCS/GraphCCS. Standardize records across datasets (applying filtering strategies similar to prior studies to mitigate inaccuracies introduced by instrument-type variation), compute RDKit 3D coordinates from SMILES, and keep columns for coordinates, smiles, adduct, and ccs. Use 2_data_splits.ipynb to generate train and test parquet files (e.g., metlin_train_3d.parquet, ccsbase_3d.parquet) for each configuration, including combined CCSBase + METLIN-CCS splits. Validate outputs by training SigmaCCS, GraphCCS, and Mol2CCS on the combined split and checking that test R2 is close to 0.95, RMSE close to 6, and MRE between 1.39% and 1.71% as reported in Fig. 4.

## Related tools

- **RDKit** (Reads SMILES representations and generates the 3D conformations stored in the curated parquet files)
- **poetry** (Installs the development environment in which the curation and splitting notebooks/scripts run)
- **Python** (Runs the data_processing notebooks and the downstream training script that consumes the splits) — https://github.com/enveda/ccs-prediction

## Examples

```
poetry run python scripts/train-test.py --prefix "train-metlin-test-ccsbase" --train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet" --coordinates-column-name "coordinates" --coordinates-present --smiles-column-name "smiles" --adduct-column-name "adduct" --ccs-column-name "ccs"
```

## Evaluation signals

- Each curated parquet contains the four expected columns: coordinates, smiles, adduct, ccs, with 3D coordinates present (or flagged absent so train-test.py generates them from SMILES)
- Train/test splits regenerated by 2_data_splits.ipynb yield combined-dataset model performance of R2 ≈ 0.95, RMSE ≈ 6, MRE between 1.39% and 1.71% (GraphCCS best), matching Fig. 4
- Cross-dataset splits (train METLIN-CCS, test CCSBase and vice versa) reproduce the reported lack of generalizability finding, confirming splits are not leaking chemical space
- Curated record counts are consistent with the source database sizes: CCSBase v1.3 with 16,989 CCS values from 6,744 molecules and METLIN-CCS with over 27,000 unique structures

## Limitations

- Variations in experimental conditions such as instrument types likely introduced inaccuracies into the CCS training values, motivating the applied filtering strategies
- Evaluation is restricted in scope mainly to small molecules and specific regions of chemical space (METLIN-CCS is based on synthetic structures)
- Each user must download the raw databases themselves under their respective licenses before running the curation notebooks

## Evidence

- [methods] data_processing notebooks format databases and make splits: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
- [methods] download raw databases and read into notebooks: "Each user should download the raw database (as excel/csv) and read them in the two notebooks for each database"
- [methods] Mol2CCS input formatting: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [readme] train-test.py input files and column arguments: "--train-input-file "ccs-prediction/metlin_train_3d.parquet" --test-input-file "ccs-prediction/ccsbase_3d.parquet""
- [discussion] filtering to mitigate training data inaccuracies: "We applied filtering strategies similar to those used in previous studies (see Section."Standardization across datasets") to try and mitigate this issue"
- [intro] CCSBase composition: "CCSBase (v1.3) combines 22 different datasets, providing a total of 16,989 CCS values measured on three different instruments from 6,744 distinct molecules"
