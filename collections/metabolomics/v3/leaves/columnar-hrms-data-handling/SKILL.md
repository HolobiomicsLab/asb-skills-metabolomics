---
name: columnar-hrms-data-handling
description: Use when HRMS data has been converted to Parquet by arcMS and you need
  to compute TIC/BPI/EIC chromatograms, MS spectra or IMS traces from the file on-disk
  without loading it fully into memory.
when_to_use_negative:
- When the whole dataset is small enough to hold in memory and full row access is
  needed — then arrow::read_parquet() to load data directly is simpler
- When the source data is still in Waters UNIFI, mzML or HDF5 format and has not yet
  been converted to Parquet with arcMS
- When native binary MS formats (e.g. vendor RAW or mzML) are required by downstream
  tools that cannot read columnar tables
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: arcMS
  role: R package that converts Waters UNIFI (HD)MSE data (and mzML) into Parquet
    columnar files that this skill queries
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Provides open_dataset() for on-disk Parquet access and dplyr-backed filter/group_by/summarise
    queries
- name: R
  role: Language in which the dplyr/Arrow aggregation pipelines are written
- name: DuckDB
  role: Backend reached via to_duckdb() to speed up binned rt/mz and IMS-trace aggregations
    to a few seconds
- name: arcms-dataviz
  role: R/Shiny companion app that visualizes Parquet data as 2D plots, TIC, BPI or
    EIC chromatograms using the same on-disk Arrow approach
  repo: https://github.com/leesulab/arcms-dataviz
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:525396d7c6fd59858bec6258d2138bce53d4066b16a59275fec24ea2a97fe425
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - DuckDB
  - R
  - arcMS
  - arcms-dataviz
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# columnar-hrms-data-handling

## Summary

Query, filter and aggregate high-resolution mass spectrometry data stored as columnar Apache Parquet files (converted with arcMS) using on-disk Arrow/dplyr (or DuckDB) operations, so chromatograms and spectra are extracted without loading the full ~1.9 Gb dataset into RAM.

## When to use

Use when HRMS data has been converted to Parquet by arcMS and you need to compute TIC/BPI/EIC chromatograms, MS spectra or IMS traces from the file on-disk without loading it fully into memory.

## When NOT to use

- When the whole dataset is small enough to hold in memory and full row access is needed — then arrow::read_parquet() to load data directly is simpler
- When the source data is still in Waters UNIFI, mzML or HDF5 format and has not yet been converted to Parquet with arcMS
- When native binary MS formats (e.g. vendor RAW or mzML) are required by downstream tools that cannot read columnar tables

## Inputs

- Apache Parquet HRMS file converted with arcMS (columns such as rt, mz, intensity, mslevel, scanid, bin), e.g. cal1.parquet downloaded from arcms-dataviz
- Optionally the associated JSON metadata file (sample-metadata.)

## Outputs

- R data frame with the aggregated result, e.g. a TIC table of two columns (rt, int) of ~64 Kb
- EIC, BPI, MS spectrum or IMS trace tables (e.g. rt/mz binned matrices of several Mb)

## How to apply

Open the Parquet file with arrow::open_dataset() to create a FileSystemDataset pointer (e.g. cal1.parquet, ~1.9 Gb when loaded) instead of read_parquet(). Build a dplyr pipeline directly on the Arrow dataset: filter(mslevel == "1") to keep low collision energy MS1 scans, then group and summarise — e.g. group_by(rt) |> summarise(int = sum(intensity)) |> collect() for a TIC, or filter(mz > 216.0016 & mz < 216.2016) with group_by(rt, scanid) for an EIC of atrazine. For heavier binned aggregations (rt_binned = floor(rt/0.1)*0.1, mz_binned = floor(mz/1)*1) or IMS traces, pipe to_duckdb() so the query runs in a few seconds instead of ~30s–1 min. Only the small aggregated result (e.g. a ~64 Kb TIC obtained in ~1s) is collected into an R data frame; full profile-mode IMS HDMSe Parquet files are ~200-400 MB on disk.

## Related tools

- **arcMS** (R package that converts Waters UNIFI (HD)MSE data (and mzML) into Parquet columnar files that this skill queries) — https://github.com/leesulab/arcMS
- **Arrow** (Provides open_dataset() for on-disk Parquet access and dplyr-backed filter/group_by/summarise queries)
- **R** (Language in which the dplyr/Arrow aggregation pipelines are written)
- **DuckDB** (Backend reached via to_duckdb() to speed up binned rt/mz and IMS-trace aggregations to a few seconds)
- **arcms-dataviz** (R/Shiny companion app that visualizes Parquet data as 2D plots, TIC, BPI or EIC chromatograms using the same on-disk Arrow approach) — https://github.com/leesulab/arcms-dataviz

## Examples

```
TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- TIC output contains exactly two columns (rt, int) with positive intensity values spanning the full retention-time range
- Collected result size is on the order of tens of Kb (~64 Kb for the example TIC), not the full ~1.9 Gb of loaded data
- Query completes in around 1s for the plain TIC; binned rt/mz queries via DuckDB finish in a few seconds
- Only MS1 (mslevel == "1", low collision energy) scans contribute to the TIC aggregation
- EIC m/z window matches the intended analyte (e.g. 216.0016 < mz < 216.2016 for atrazine)

## Limitations

- Pure Arrow queries on rt/mz binned aggregations take ~30s to 1 min; DuckDB via to_duckdb() is needed for few-second runtime
- HDF5 output has larger filesize than Parquet, and HDF5 files split low/high energy into separate ms1/ms2 datasets
- No changelog is available for the arcMS repository, so version changes to data schema or functions are hard to track

## Evidence

- [methods] On-disk Arrow TIC query: "TIC = data_arrow |>
  filter(mslevel == "1") |>
  group_by(rt) |> 
  summarise(int = sum(intensity)) |> 
  collect()"
- [methods] On-disk query speed and size: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] In-memory size of example data: "the loaded data for this example sample occupies \~1.9 Gb in memory"
- [methods] Binned query timing and DuckDB acceleration: [evidence span withheld — anchor status: not_found]
- [readme] README: on-disk visualization rationale: "Data is not fully loaded in RAM but chromatograms and MS spectra are quickly retrieved from the parquet file with the [Apache Arrow
library]"
- [readme] README: output formats: "the [Apache Parquet](https://parquet.apache.org/) format for minimal
  filesize and fast access."
