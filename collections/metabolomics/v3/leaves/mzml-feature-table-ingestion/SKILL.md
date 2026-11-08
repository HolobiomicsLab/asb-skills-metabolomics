---
name: mzml-feature-table-ingestion
description: Use when you have raw LC-MS mzML files plus an aligned or unaligned feature
  table CSV (from mzMine2, XCMS, etc.) and need to load both into NeatMS before peak
  prediction or filtering.
when_to_use_negative:
- If you already have a labelled/filtered peak table and only need to re-export it
  — ingestion into a new Experiment would duplicate work rather than relabel peaks.
- If your feature table comes from a pipeline whose column schema NeatMS does not
  recognise (e.g. a custom exporter not matching mzMine2 or XCMS formats) — the Experiment
  constructor will not map the properties correctly.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: NeatMS
  role: Python package providing the Experiment class that ingests raw mzML data and
    pipeline-produced feature tables for subsequent peak labelling and filtering
  repo: https://github.com/bihealth/NeatMS
- name: pandas
  role: CSV feature tables produced via pandas' csv export method are the ingestion
    format NeatMS reads
- name: mzMine2
  role: Upstream pipeline whose exported aligned/unaligned feature CSV is ingested
    as a NeatMS feature table
- name: XCMS
  role: Alternative upstream pipeline whose feature table (supported from NeatMS 0.7
    for aligned peaks) can be ingested
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02220
    title: Deep Learning-Assisted Peak Curation for Large-Scale LC-MS Metabolomics
  source_content_hash: sha256:3693449eb1db9d576b2469226caf3f5f4bdc3e2b0d6eed19c37796894ddaf759
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
  - Tensorflow
  - XCMS
  - mzMine2
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mzml-feature-table-ingestion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ingest raw mzML files and a commonly produced feature table (e.g. from mzMine2 or XCMS) into a NeatMS Experiment object, so neural-network peak labelling and filtering can be applied downstream. This is the data-loading step that maps aligned/unaligned feature CSVs onto the raw spectral data.

## When to use

Use when you have raw LC-MS mzML files plus an aligned or unaligned feature table CSV (from mzMine2, XCMS, etc.) and need to load both into NeatMS before peak prediction or filtering.

## When NOT to use

- If you already have a labelled/filtered peak table and only need to re-export it — ingestion into a new Experiment would duplicate work rather than relabel peaks.
- If your feature table comes from a pipeline whose column schema NeatMS does not recognise (e.g. a custom exporter not matching mzMine2 or XCMS formats) — the Experiment constructor will not map the properties correctly.

## Inputs

- folder of raw mzML files (raw_data_folder_path)
- feature table CSV from a common processing pipeline (aligned_features.csv or unaligned_features folder), e.g. from mzMine2 or XCMS
- input_data keyword identifying the pipeline ('mzmine')

## Outputs

- NeatMS Experiment object holding samples and peak objects
- NN_handler bound to the experiment, ready for model loading and peak prediction

## How to apply

Build a NeatMS Experiment with ntms.Experiment(raw_data_folder_path, feature_table_path, input_data) where raw_data_folder_path points at the mzML folder and feature_table_path is the aligned (or unaligned) CSV; set input_data='mzmine' (or the corresponding pipeline keyword) to tell NeatMS which pipeline produced the table. The default model expects the standard set of peak properties (rt, mz, height, area) in the feature table, as exemplified by the repo's data/test_data/aligned_features.csv. After the Experiment is created, instantiate an NN_handler on it and load the pretrained model with nn_handler.create_model(model='path_to_model.h5'). This ingestion step must succeed before predict_peaks or export_csv is called, since both operate on the Experiment object's peak metadata.

## Related tools

- **NeatMS** (Python package providing the Experiment class that ingests raw mzML data and pipeline-produced feature tables for subsequent peak labelling and filtering) — https://github.com/bihealth/NeatMS
- **pandas** (CSV feature tables produced via pandas' csv export method are the ingestion format NeatMS reads)
- **mzMine2** (Upstream pipeline whose exported aligned/unaligned feature CSV is ingested as a NeatMS feature table)
- **XCMS** (Alternative upstream pipeline whose feature table (supported from NeatMS 0.7 for aligned peaks) can be ingested)

## Examples

```
import NeatMS as ntms; experiment = ntms.Experiment('mzml_folder/', 'aligned_features.csv', input_data='mzmine')
```

## Evaluation signals

- Experiment object is created without error and contains one Sample per mzML file in raw_data_folder_path.
- Every peak in the feature table is present in the Experiment and carries the standard properties rt, mz, height, area.
- input_data keyword matches the pipeline that produced the CSV ('mzmine' for mzMine2/XCMS-derived tables), otherwise column mapping will silently fail.
- Downstream nn_handler.predict_peaks produces labels (High_quality, Low_quality, Noise) for all peaks, confirming ingestion succeeded.
- Feature table columns match the expected schema (cf. aligned_features.csv example in the repo).

## Limitations

- The ingestion constructor relies on a specific feature-table column schema; custom pipelines require manual column renaming to match NeatMS expectations.
- Aligned peak support for XCMS was only added from NeatMS version 0.7 — older versions cannot ingest XCMS aligned tables.
- Peaks that are labelled Noise are exported as missing values, so ingestion+export does not preserve noise-labelled entries explicitly.

## Evidence
