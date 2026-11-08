---
name: lcms-feature-table-import
description: Use when you have raw mzML files plus an mzMine- or XCMS-derived feature
  table CSV/TSV and need to bring both into a single NeatMS Experiment for downstream
  peak labelling.
when_to_use_negative:
- Input is only an already-predicted or annotation-labelled peak table that skips
  the curation step.
- The raw LC-MS data folder is unavailable or unreadable — the Experiment constructor
  loads both raw data and features, so one cannot be supplied without the other.
- The feature table comes from a pipeline other than mzMine or XCMS (no parser is
  documented in this context).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NeatMS
  role: Provides the Experiment object (ntms.Experiment(raw_data_folder_path, feature_table_path,
    input_data)) that ingests both the raw mzML files and the mzMine/XCMS feature
    table in one call
  repo: https://github.com/bihealth/NeatMS
- name: mzMine2
  role: Upstream peak-detection tool whose exported aligned_features.csv serves as
    the feature_table_path input
- name: XCMS
  role: Alternative upstream peak-detection tool whose output can be imported when
    input_data='xcms' (supported for aligned peaks from NeatMS version 0.7)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c02220
    title: Deep Learning-Assisted Peak Curation for Large-Scale LC-MS Metabolomics
  source_content_hash: sha256:afd1534364bf37929a3c67122a4e1092706cf225ed490afe3dbc8d05c85b00a8
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
  - XCMS
  - mzMine2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# lcms-feature-table-import

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load raw LC-MS data files and a processed feature table (e.g. mzMine or XCMS CSV output) into a NeatMS Experiment object, which builds per-sample peak/feature collections ready for neural-network peak curation.

## When to use

Use when you have raw mzML files plus an mzMine- or XCMS-derived feature table CSV/TSV and need to bring both into a single NeatMS Experiment for downstream peak labelling.

## When NOT to use

- Input is only an already-predicted or annotation-labelled peak table that skips the curation step.
- The raw LC-MS data folder is unavailable or unreadable — the Experiment constructor loads both raw data and features, so one cannot be supplied without the other.
- The feature table comes from a pipeline other than mzMine or XCMS (no parser is documented in this context).

## Inputs

- Folder of raw LC-MS files (raw_data_folder_path)
- Aligned feature table CSV from mzMine2 (e.g. aligned_features.csv)
- Unaligned feature tables (alternative input, unaligned_features folder)
- input_data string selecting the pipeline parser ('mzmine' or 'xcms')

## Outputs

- NeatMS Experiment object (exp) with exp.feature_tables[0].feature_collection_list populated
- Feature collections (one per feature) accessible for peak-property inspection and later nn_handler.predict_peaks()

## How to apply

Set three arguments before creating the Experiment object: raw_data_folder_path pointing to the folder containing the raw LC-MS files, feature_table_path pointing to the processed feature table (aligned_features.csv exported from mzMine2, or the mzML/TSV output expected by NeatMS 0.7+ for XCMS), and input_data set to 'mzmine' or 'xcms' to select the parser. Instantiate with ntms.Experiment(raw_data_folder_path, feature_table_path, input_data); NeatMS then automatically loads the raw data and the features, building exp.feature_tables[0].feature_collection_list. Validate the import by inspecting the feature collections via the tutorial's Counter-based exploration snippet; for the NeatMS tutorial dataset this should yield 13,852 feature collections.

## Related tools

- **NeatMS** (Provides the Experiment object (ntms.Experiment(raw_data_folder_path, feature_table_path, input_data)) that ingests both the raw mzML files and the mzMine/XCMS feature table in one call) — https://github.com/bihealth/NeatMS
- **mzMine2** (Upstream peak-detection tool whose exported aligned_features.csv serves as the feature_table_path input)
- **XCMS** (Alternative upstream peak-detection tool whose output can be imported when input_data='xcms' (supported for aligned peaks from NeatMS version 0.7))

## Examples

```
import NeatMS as ntms
experiment = ntms.Experiment(raw_data_folder_path, feature_table_path, input_data)
```

## Evaluation signals

- exp.feature_tables[0].feature_collection_list is populated and non-empty after construction.
- For the NeatMS tutorial dataset, the number of feature collections counted via the Counter snippet equals 13,852, matching the value printed in the basic-usage tutorial.
- input_data string matches the actual provenance of the feature table ('mzmine' or 'xcms'); a mismatch would produce a parsing error or empty collections.
- Raw data files referenced by the feature table are found in raw_data_folder_path; missing samples should surface as load errors or absent sample objects.

## Limitations

- Aligned XCMS peaks are only supported from NeatMS version 0.7 onward; older versions cannot import XCMS aligned tables.
- NeatMS itself gives no information on peak identity — imported features remain unlabelled metabolite peaks until predicted via the neural network handler.
- The tool cannot remove peaks from non-lipid metabolites unless provided with a curated library panel of metabolites of interest.

## Evidence

- [methods] Set the paths to the tutorial raw data folder and the mzMine feature table (aligned_features.csv from the NeatMS repository test data) and set input_data='mzmine'.: [evidence span withheld — anchor status: not_found]
- [methods] Create the NeatMS Experiment object with ntms.Experiment(raw_data_folder_path, feature_table_path, input_data), which automatically loads the raw data and features.: [evidence span withheld — anchor status: not_found]
- [readme] The source code and related materials (e.g. tutorials, example data, neural network model) are available at https://github.com/bihealth/NeatMS.: "The source code and related materials (e.g. tutorials, example data, neural network model) are available at [https://github.com/bihealth/NeatMS](https://github.com/bihealth/NeatMS)."
- [methods] example of a single file containing aligned peaks is available here: "example of a single file containing aligned peaks is available [here](https://github.com/bihealth/NeatMS/blob/master/data/test_data/aligned_features.csv)"
- [methods] Update, from version 0.7 aligned peaks are also supported with XCMS: "Update, from version 0.7 aligned peaks are also supported with XCMS"
