---
name: chromatographic-data-storage-comparison
description: Use when comparing mass spectrometry data conversion pipelines or storage
  formats (e.g., Parquet vs mzML/mz5/mzMLb) on conversion time, file size, and data
  read/query speed for HRMS or IMS-HDMS^E datasets.
when_to_use_negative:
- Data are not HRMS/MSE acquisitions and no cross-format storage comparison is required
  (e.g., already a processed feature table)
- The sole goal is format conversion itself, not benchmarking conversion speed, file
  size, and read speed across formats
- Waters UNIFI API access is unavailable and no mzML or Parquet input exists, making
  the arcMS conversion leg of the benchmark impossible
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: arcMS
  role: Converts Waters UNIFI (HD)MS^E data to Apache Parquet for the fast-conversion,
    small-filesize benchmark arm
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: On-disk querying, filtering, and aggregation of Parquet HRMS data to time
    read/query speed (TIC, EIC, binned rt/mz queries)
- name: Python
  role: Cross-language read access to Parquet files demonstrating portability of the
    format
- name: R
  role: Environment for arcMS, arrow/dplyr queries, and benchmark timing
  repo: https://github.com/leesulab/arcMS
- name: Shiny
  role: Companion arcms-dataviz app demonstrating fast on-disk visualization of Parquet
    HRMS data without full RAM loading
  repo: https://github.com/leesulab/arcms-dataviz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:d512d54e5b2bc01ee7ea7b2386d4505f31dd67003ef2366247d939590ff4a7cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - Python
  - R
  - Shiny
  - arcMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# chromatographic-data-storage-comparison

## Summary

Benchmark workflow for comparing HRMS data storage formats (Apache Parquet via arcMS vs mz5, mzML, gzipped mzML, mzMLb) across conversion speed, on-disk file size, and downstream read/query speed using environmental HRMS samples.

## When to use

Use when comparing mass spectrometry data conversion pipelines or storage formats (e.g., Parquet vs mzML/mz5/mzMLb) on conversion time, file size, and data read/query speed for HRMS or IMS-HDMS^E datasets.

## When NOT to use

- Data are not HRMS/MSE acquisitions and no cross-format storage comparison is required (e.g., already a processed feature table)
- The sole goal is format conversion itself, not benchmarking conversion speed, file size, and read speed across formats
- Waters UNIFI API access is unavailable and no mzML or Parquet input exists, making the arcMS conversion leg of the benchmark impossible

## Inputs

- HRMS sample set (e.g., 10 environmental HRMS samples) in Waters UNIFI (HD)MS^E format or mzML
- Example Parquet file cal1-full-long.parquet from the leesulab/arcms-dataviz repository
- MSConvertGUI (ProteoWizard) for reference conversions to mz5, mzML, gzipped mzML, mzMLb

## Outputs

- Structured benchmark table (format, conversion time, file size, read time), e.g., benchmark_table.csv
- Timings and sizes for arcMS/Parquet vs mz5, mzML, gzipped mzML, mzMLb conversions
- Query timing measurements (TIC ~1 s / ~64 Kb via Arrow; binned aggregation Arrow ~30 s-1 min vs DuckDB a few seconds)

## How to apply

Convert a common set of HRMS samples (article used 10 environmental HRMS samples) with arcMS to Parquet and with MSConvertGUI to mz5, mzML, gzipped mzML, and mzMLb, timing each conversion (article reported arcMS at 1-3 min vs MSConvertGUI at 11-18 min, mzMLb ~7 min). Record resulting file sizes: Parquet 80-600 MB (~200-400 MB for full profile-mode IMS HDMS^E), uncompressed mzML 5-6 GB, gzipped mzML/mz5 300 MB-1 GB, mzMLb ~200-600 MB. Then time reading/querying each format on identical tasks, e.g., loading cal1-full-long.parquet with arrow::read_parquet() (~1.9 Gb in memory) and computing a TIC via an on-disk open_dataset() query filtering mslevel == '1', group_by(rt), summarise(intensity) (~1 s, ~64 Kb collected), versus mzML reads with DEIMoS (15-20 min) or MZmine (1-2 min). Also contrast a binned rt/mz aggregation (rt_binned = floor(rt/0.1)*0.1, mz_binned = floor(mz/1)*1) run via pure Arrow (~30 s-1 min) against the same query streamed through to_duckdb() (a few seconds). Assemble all measured and reported values into a table with format, conversion time, file size, and read time columns.

## Related tools

- **arcMS** (Converts Waters UNIFI (HD)MS^E data to Apache Parquet for the fast-conversion, small-filesize benchmark arm) — https://github.com/leesulab/arcMS
- **Arrow** (On-disk querying, filtering, and aggregation of Parquet HRMS data to time read/query speed (TIC, EIC, binned rt/mz queries))
- **Python** (Cross-language read access to Parquet files demonstrating portability of the format)
- **R** (Environment for arcMS, arrow/dplyr queries, and benchmark timing) — https://github.com/leesulab/arcMS
- **Shiny** (Companion arcms-dataviz app demonstrating fast on-disk visualization of Parquet HRMS data without full RAM loading) — https://github.com/leesulab/arcms-dataviz

## Examples

```
data_arrow = arrow::open_dataset("cal1-full-long.parquet"); TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- The Arrow TIC query (filter mslevel == '1', group_by rt, sum intensity, collect()) completes in approximately 1 s and returns only ~64 Kb
- arrow::read_parquet() on cal1-full-long.parquet yields an in-memory footprint of ~1.9 Gb
- Full profile-mode IMS HDMS^E Parquet file sizes fall in the ~200-400 MB range reported in the article
- Conversion times reproduce the reported gap: arcMS 1-3 min for 10 samples vs MSConvertGUI 11-18 min (mz5/mzML/gzipped mzML), ~7 min for mzMLb
- The binned rt/mz aggregation runs in a few seconds when streamed via to_duckdb(), versus ~30 s-1 min with pure Arrow

## Limitations

- Pure Arrow binned rt/mz aggregation is slow (~30 s-1 min); DuckDB streaming is needed for seconds-scale performance
- arcMS requires a properly configured Waters UNIFI Web API (connection parameters, client app registration), or mzML input via convert_mzml_to_parquet()
- HDF5 output offers fast access but larger file sizes than Parquet; benchmark conclusions are specific to the article's HRMS/IMS-HDMS^E sample set

## Evidence

- [results] conversion speed and read speed benchmark: [evidence span withheld — anchor status: not_found]
- [methods] on-disk Arrow TIC query performance: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] in-memory footprint of example Parquet file: "the loaded data for this example sample occupies \~1.9 Gb in memory"
- [methods] profile-mode IMS HDMS^E Parquet file size: "A full data file in profile mode containing IMS, and HDMS<sup>E</sup> data (MS and MS/MS), is around 200-400 Mo."
- [methods] Arrow vs DuckDB binned aggregation timing: [evidence span withheld — anchor status: not_found]
- [readme] Parquet output rationale: "the [Apache Parquet](https://parquet.apache.org/) format for minimal
  filesize and fast access."
