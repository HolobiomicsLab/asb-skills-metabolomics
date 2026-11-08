---
name: feature-collection-counting
description: Use when loading an mzMine or XCMS feature table into NeatMS and you
  need to confirm the expected total number of feature collections before filtering
  or neural-network labelling.
when_to_use_negative:
- The feature table has not been loaded through a NeatMS Experiment object (e.g. raw
  CSV pandas import) — the feature_collection_list attribute will not exist.
- The goal is to count features after NeatMS filtering (export_classes/min_group_size)
  rather than the pre-filter total.
- Input is a non-mzMine/non-XCMS format that the NeatMS Experiment parser does not
  support.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: NeatMS
  role: Provides the Experiment object whose feature_tables[0].feature_collection_list
    is counted; also ships the tutorial raw-data and aligned_features.csv test data
    used as inputs.
  repo: https://github.com/bihealth/NeatMS
- name: mzMine2
  role: Source of the aligned feature CSV (aligned_features.csv) that NeatMS ingests
    with input_data='mzmine'.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c02220
    title: Deep Learning-Assisted Peak Curation for Large-Scale LC-MS Metabolomics
  source_content_hash: sha256:dac1217e9f0c400aa90269ea9ed09870591cc5b9915ff5c4e7a1c7268b26fd89
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/bihealth/NeatMS
  repo_url: https://github.com/bihealth/NeatMS
  tools:
  - NeatMS
  - mzMine2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# feature-collection-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Enumerates the number of peak/feature collections loaded from an mzMine feature table (e.g. aligned_features.csv) into a NeatMS Experiment, yielding a per-table total used as a tutorial-validation sanity check.

## When to use

Use when loading an mzMine or XCMS feature table into NeatMS and you need to confirm the expected total number of feature collections before filtering or neural-network labelling.

## When NOT to use

- The feature table has not been loaded through a NeatMS Experiment object (e.g. raw CSV pandas import) — the feature_collection_list attribute will not exist.
- The goal is to count features after NeatMS filtering (export_classes/min_group_size) rather than the pre-filter total.
- Input is a non-mzMine/non-XCMS format that the NeatMS Experiment parser does not support.

## Inputs

- mzMine aligned feature table CSV (e.g. aligned_features.csv from NeatMS test data)
- folder of raw LC-MS data files
- input_data format flag ('mzmine' or 'xcms')

## Outputs

- integer count of feature collections (written to feature_collection_count.txt)

## How to apply

1. Point NeatMS at the raw-data folder and the mzMine feature table (aligned_features.csv from the NeatMS repository test data) with input_data='mzmine'. 2. Instantiate ntms.Experiment(raw_data_folder_path, feature_table_path, input_data), which automatically loads both the raw data and the features. 3. Access exp.feature_tables[0].feature_collection_list and count entries, e.g. via collections.Counter over the collection list as shown in the NeatMS basic-usage tutorial notebook (notebook/tutorial/NeatMS.ipynb). 4. Write the resulting total to a file such as feature_collection_count.txt. Rationale: the count is the tutorial's expected value (13,852) and verifies that the Experiment object parsed the aligned peak table correctly before any peak prediction or filtering step (e.g. nn_handler.predict_peaks(threshold=0.22)) is attempted.

## Related tools

- **NeatMS** (Provides the Experiment object whose feature_tables[0].feature_collection_list is counted; also ships the tutorial raw-data and aligned_features.csv test data used as inputs.) — https://github.com/bihealth/NeatMS
- **mzMine2** (Source of the aligned feature CSV (aligned_features.csv) that NeatMS ingests with input_data='mzmine'.)

## Examples

```
import NeatMS as ntms; exp = ntms.Experiment(raw_data_folder_path, 'aligned_features.csv', 'mzmine'); print(len(exp.feature_tables[0].feature_collection_list))
```

## Evaluation signals

- Total count equals 13,852 — the value printed in the NeatMS basic-usage tutorial.
- The count is non-zero and matches len(exp.feature_tables[0].feature_collection_list).
- The output file feature_collection_count.txt contains exactly one integer.
- Counting is done before any nn_handler.predict_peaks or export_csv call, so the number reflects the unfiltered table.
- The aligned_features.csv path resolves to the NeatMS test-data file referenced in the tutorial.

## Limitations

- The count is specific to the tutorial's test dataset (aligned_features.csv); other mzMine exports will yield different totals.
- Only MS1 features reported by mzMine2 or XCMS (and only from NeatMS ≥0.7 for XCMS aligned peaks) are loadable; other pipeline formats are not supported.
- The count is a pre-filtering figure — chemical noise peaks are still included, so it overestimates features that would survive NeatMS labelling.

## Evidence

- [readme] The source code and related materials (e.g. tutorials, example data, neural network model) are available at https://github.com/bihealth/NeatMS.: "The source code and related materials (e.g. tutorials, example data, neural network model) are available"
- [methods] Experiment object construction loads raw data and features automatically.: "experiment = ntms.Experiment(raw_data_folder_path, feature_table_path, input_data)"
- [methods] Aligned mzMine peak table used as the tutorial input.: "example of a single file containing aligned peaks is available"
- [methods] Tutorials provide the exploration snippet and expected count.: "tutorials provided as jupyter notebooks on NeatMS github repository"
