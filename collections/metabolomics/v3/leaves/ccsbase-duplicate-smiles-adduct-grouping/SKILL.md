---
name: ccsbase-duplicate-smiles-adduct-grouping
description: Use when preparing CCSBase for CCS-prediction model training and the
  loaded table contains multiple records sharing the same SMILES and adduct, requiring
  deduplication driven by a deviation-based consistency threshold.
when_to_use_negative:
- Input is already a deduplicated dataset or a single-record-per-SMILES-adduct table
  (e.g. a prepared train/test split such as metlin_train_3d.parquet) — grouping would
  be a no-op
- You are deduplicating METLIN-CCS rather than CCSBase — the paper applies similar
  standardization across datasets, but this specific mean-absolute-deviation > 5 rule
  was derived from histogram analysis of CCSBase duplicates
- You need to filter model predictions (e.g. confidence-model-based filtering of predicted
  CCS) rather than clean experimental training values
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: Python
  role: Implements the pandas grouping, mean-absolute-deviation computation, threshold
    filtering, and parquet export in the data_processing notebooks
  repo: https://github.com/enveda/ccs-prediction
- name: poetry
  role: Environment/dependency management for running the repository's data-processing
    and training scripts
  repo: https://github.com/enveda/ccs-prediction
- name: Mol2CCS
  role: 'Downstream consumer: the deduplicated CCSBase table is formatted according
    to the input of Mol2CCS for GNN training'
  repo: https://github.com/enveda/ccs-prediction
- name: RDKit (v2023.09.5)
  role: Reads the SMILES representation of each molecule during data processing/standardization
  repo: https://www.rdkit.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-024-00899-w
    title: Evaluating the generalizability of graph neural networks for predicting
      collision cross section
  source_content_hash: sha256:0941d741c911bbf14437eb0334b9ba6d92faca8b792e04069d602ef863fdda51
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ccsbase-duplicate-smiles-adduct-grouping

## Summary

Standardizes CCSBase (v1.3) by grouping entries on SMILES-adduct pairs, flagging duplicate groups whose mean absolute CCS deviation exceeds 5 for removal, and collapsing remaining duplicates to a single mean-CCS record. This removes inconsistent experimental replicate values before GNN (SigmaCCS/GraphCCS/Mol2CCS) training.

## When to use

Use when preparing CCSBase for CCS-prediction model training and the loaded table contains multiple records sharing the same SMILES and adduct, requiring deduplication driven by a deviation-based consistency threshold.

## When NOT to use

- Input is already a deduplicated dataset or a single-record-per-SMILES-adduct table (e.g. a prepared train/test split such as metlin_train_3d.parquet) — grouping would be a no-op
- You are deduplicating METLIN-CCS rather than CCSBase — the paper applies similar standardization across datasets, but this specific mean-absolute-deviation > 5 rule was derived from histogram analysis of CCSBase duplicates
- You need to filter model predictions (e.g. confidence-model-based filtering of predicted CCS) rather than clean experimental training values

## Inputs

- Processed CCSBase (v1.3) table as parquet or CSV, with 'smiles', 'adduct', and 'ccs' columns (raw database from https://ccsbase.net/query formatted per the data_processing notebooks)

## Outputs

- 'ccsbase_deduplicated.parquet' — a deduplicated CCSBase table with 'ccs', 'smiles', and 'adduct' columns, where retained duplicate SMILES-adduct pairs have mean absolute CCS deviation ≤ 5 and remaining duplicates carry the mean CCS value

## How to apply

Load the processed CCSBase table (parquet/CSV) into pandas and group records by the SMILES-adduct pair, keeping only groups with more than one record as duplicates. For each duplicate pair, compute the mean absolute deviation of the CCS values; the threshold of 5 was chosen through histogram analysis of the deviations across all duplicate groups in the paper's standardization. Drop duplicate pairs whose mean absolute deviation exceeds 5 (inconsistent replicate measurements), and for retained duplicates assign the mean CCS value to a single deduplicated record. Save the result with the CCS column ('ccs'), SMILES ('smiles'), and adduct ('adduct') columns as 'ccsbase_deduplicated.parquet' for downstream data splits and model training via the enveda/ccs-prediction notebooks.

## Related tools

- **Python** (Implements the pandas grouping, mean-absolute-deviation computation, threshold filtering, and parquet export in the data_processing notebooks) — https://github.com/enveda/ccs-prediction
- **poetry** (Environment/dependency management for running the repository's data-processing and training scripts) — https://github.com/enveda/ccs-prediction
- **Mol2CCS** (Downstream consumer: the deduplicated CCSBase table is formatted according to the input of Mol2CCS for GNN training) — https://github.com/enveda/ccs-prediction
- **RDKit (v2023.09.5)** (Reads the SMILES representation of each molecule during data processing/standardization) — https://www.rdkit.org/

## Examples

```
poetry run python -c "import pandas as pd; df=pd.read_parquet('ccsbase.parquet'); g=df.groupby(['smiles','adduct'])['ccs'].agg(['mean','count']); df=df.merge(g[['mean']],on=['smiles','adduct']); df=df[df['ccs'].sub(df['mean']).abs().groupby([df.smiles,df.adduct]).transform('mean')<=5]; df=df.drop_duplicates(['smiles','adduct']); df.to_parquet('ccsbase_deduplicated.parquet')"
```

## Evaluation signals

- Output file 'ccsbase_deduplicated.parquet' exists and is a valid parquet with exactly the 'ccs', 'smiles', 'adduct' columns
- No SMILES-adduct group retains more than one record after deduplication
- Every retained record that came from a duplicate pair has mean absolute CCS deviation ≤ 5
- Removed duplicate pairs all had mean absolute CCS deviation > 5 (can be verified by re-plotting the deviation histogram of the dropped groups)
- Row count of the deduplicated table equals unique SMILES-adduct groups surviving the threshold from the original processed table

## Limitations

- The threshold of 5 is empirical, derived from histogram analysis of CCSBase duplicate deviations; it may not transfer to other CCS databases or instrument mixes
- Variations in experimental conditions such as instrument types likely introduced inaccuracies in CCS training values — deduplication by mean absolute deviation mitigates but does not eliminate cross-instrument inconsistency
- The paper notes filtering strategies were applied to mitigate inaccuracies in experimental CCS training data, yet models trained on CCSBase still failed to generalize to other chemical spaces, so this cleaning step alone does not guarantee model generalizability

## Evidence

- [methods] Duplicate SMILES-adduct pairs in CCSBase were filtered by removing any pairs whose mean absolute deviation of CCS values was greater than five, a threshold determined through histogram analysis: [evidence span withheld — anchor status: not_found]
- [methods] Each notebook reads the csv/excel and formats it according to the input of Mol2CCS.: "Each notebook reads the csv/excel and formats it according to the input of Mol2CCS."
- [methods] data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits.: "data_processing: notebooks to process the METLIN-CCS and CCSBase and make the data splits."
- [discussion] variations in the experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values: "variations in the experimental conditions, such as instrument types, likely introduced some inaccuracies in the CCS values used to train our models"
- [intro] we first read the SMILES representation of each molecule using RDKit (v2023.09.5): "we first read the SMILES representation of each molecule using RDKit (v2023.09.5)"
