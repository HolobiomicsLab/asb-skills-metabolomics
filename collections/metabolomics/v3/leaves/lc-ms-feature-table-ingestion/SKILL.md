---
name: lc-ms-feature-table-ingestion
description: Use when starting an mpactR workflow with an exported peak table (e.g.
  Progenesis CSV with compound, m/z, retention time, and raw abundance columns) and
  a companion sample metadata CSV that must be loaded into a single R object before
  applying filters.
when_to_use_negative:
- The input is not a pre-processed feature/peak table but raw vendor or mzML spectrometry
  data — mpactR filters start from pre-processed tandem MS/MS output
- The peak table was exported from a pre-processor other than Progenesis (e.g. MS-DIAL
  or Bruker Metaboscape exports are supported in the original Python MPACT, not documented
  for mpactR's import_data)
- The metadata CSV lacks the required Injection, Sample_Code, and Biological_Group
  columns needed for group-based filtering
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mpactr
  role: Provides import_data() to build the R6 object from peak table + metadata,
    plus example_path(), get_peak_table(), get_metadata(), and get_raw_data() for
    validation
  repo: https://github.com/mums2/mpactr
- name: R
  role: Runtime environment; import_data() and its accessors return data.table objects
    consumed in an R pipeline
  repo: https://github.com/mums2/mpactr/actions/workflows/r.yml
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/mra.00997-24
    title: 'mpactR: an R adaptation of the metabolomics peak analysis computational
      tool (MPACT) for use in reproducible data analysis pipelines'
  source_content_hash: sha256:6e02f2f42cba4ffc669ae855bac09c558b6260add207748146be9bbd90d2395b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mums2/mpactr
  tools:
  - R
  - mpactr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lc-ms-feature-table-ingestion

## Summary

Ingest LC-MS/MS pre-processed peak tables (e.g. Progenesis-format CSV exports) together with sample metadata into mpactR's R6 class object via import_data(), the required first stage of any mpactR filtering pipeline. This step unifies compound/m/z/retention-time/abundance data with biological group annotations so downstream filters can operate on both.

## When to use

Use when starting an mpactR workflow with an exported peak table (e.g. Progenesis CSV with compound, m/z, retention time, and raw abundance columns) and a companion sample metadata CSV that must be loaded into a single R object before applying filters.

## When NOT to use

- The input is not a pre-processed feature/peak table but raw vendor or mzML spectrometry data — mpactR filters start from pre-processed tandem MS/MS output
- The peak table was exported from a pre-processor other than Progenesis (e.g. MS-DIAL or Bruker Metaboscape exports are supported in the original Python MPACT, not documented for mpactR's import_data)
- The metadata CSV lacks the required Injection, Sample_Code, and Biological_Group columns needed for group-based filtering

## Inputs

- Progenesis-format CSV peak table (compound, m/z, retention time, and raw abundance columns per sample)
- CSV sample metadata file with required columns Injection, Sample_Code, and Biological_Group

## Outputs

- mpactR R6 class object holding both the raw peak table and metadata (e.g. saved as imported_data.rds)
- Peak table retrievable via get_peak_table() / get_raw_data() as a data.table
- Metadata retrievable via get_metadata() as a data.table

## How to apply

Call mpactR::import_data() with its two file-path arguments, peak_table_file_path and metadata_file_path, and set format = 'Progenesis' when the peak table was exported from Progenesis (the supported pre-processing software format). mpactR bundles example files (cultures_peak_table.csv and cultures_metadata.csv) that can be resolved with example_path() to smoke-test the ingestion; the example metadata must contain the required columns Injection, Sample_Code, and Biological_Group. The call returns an R6 class object storing both the peak table and the metadata; retrieve them with get_peak_table() and get_metadata(), which return data.table objects, and get_raw_data() to confirm the unfiltered Progenesis table is intact. Only after validating the imported object should downstream filters (filter_mispicked_ions(), filter_group(), filter_cv(), filter_insource_ions()) be applied.

## Related tools

- **mpactr** (Provides import_data() to build the R6 object from peak table + metadata, plus example_path(), get_peak_table(), get_metadata(), and get_raw_data() for validation) — https://github.com/mums2/mpactr
- **R** (Runtime environment; import_data() and its accessors return data.table objects consumed in an R pipeline) — https://github.com/mums2/mpactr/actions/workflows/r.yml

## Examples

```
data <- mpactr::import_data(peak_table_file_path = "cultures_peak_table.csv", metadata_file_path = "cultures_metadata.csv", format = "Progenesis")
```

## Evaluation signals

- get_metadata(data) returns a data.table containing the columns Injection, Sample_Code, and Biological_Group
- get_raw_data(data) returns the unfiltered Progenesis peak table with compound, m/z, retention time, and raw abundance columns
- get_peak_table(data) retrieves the stored peak table from the R6 object (e.g. get_peak_table(data2))
- The returned object is an mpactR R6 class instance and can be serialized, e.g. saved to imported_data.rds for the next pipeline stage

## Limitations

- Only pre-processed data are handled: mpactR corrects errors that occur during pre-processing of raw tandem MS/MS data, so raw vendor files cannot be ingested directly
- The Progenesis format is the documented import format; other pre-processor exports are not described for mpactR's import_data()
- Ingestion does not itself correct mispicked peaks, blank carryover, non-reproducible ions, or in-source fragments — the independent filters must be run afterwards

## Evidence

- [methods] To import these data into R, use the mpactR function `import_data()`: "To import these data into R, use the mpactR function `import_data()`"
- [methods] Metadata can be accessed with `get_metadata()`: "Metadata can be accessed with `get_metadata()`"
- [abstract] mpactr automates pre-processing of liquid chromatography-tandem mass spectrometry (LC-MS/MS) data from microbiological samples: [evidence span withheld — anchor status: not_found]
- [methods] The goal of mpactr is to correct for errors that occur during the pre-processing of raw tandem MS/MS data: "The goal of mpactr is to correct for errors that occur during the pre-processing of raw tandem MS/MS data"
