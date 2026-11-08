---
name: parquet-on-disk-query-timing-measurement
description: Use when you need to benchmark read/query performance of Parquet-stored
  HRMS (MSE/HDMSE) data against other formats (mzML, mz5, mzMLb), measuring query
  wall time and output data volume from on-disk Arrow datasets.
when_to_use_negative:
- Input data is not in Parquet columnar format (e.g. proprietary UNIFI raw data not
  yet converted with arcMS, or mzML/HDF5 files) — convert first or use format-specific
  readers
- The goal is conversion-speed benchmarking of MSConvertGUI vs arcMS rather than on-disk
  query/read timing of already-converted files
- The query fully loads the file into memory via read_parquet(), since then no on-disk
  lazy evaluation is being measured
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Arrow
  role: Used to open the Parquet file on disk with open_dataset() and run lazy dplyr
    filter/group_by/summarise queries (TIC, EIC, binned rt/mz) without loading all
    data into RAM
  repo: https://github.com/leesulab/arcMS
- name: arcMS
  role: R package that converts Waters UNIFI (HD)MSE data to the Parquet files that
    this timing measurement queries
  repo: https://github.com/leesulab/arcMS
- name: R
  role: Environment in which the timing benchmark dplyr/Arrow/DuckDB queries are executed
    and timed
  repo: https://github.com/leesulab/arcMS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:03d6eabb2ce2b6e85c356bad9883a5726796c6310b0f9117942d33a94c776572
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - R
  - arcMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# parquet-on-disk-query-timing-measurement

## Summary

A benchmarking procedure for quantifying how fast HRMS data stored in Apache Parquet can be queried on disk (TIC, binned rt/mz aggregation) using Arrow and DuckDB, contrasting in-memory footprint with lazy query time and output size.

## When to use

Use when you need to benchmark read/query performance of Parquet-stored HRMS (MSE/HDMSE) data against other formats (mzML, mz5, mzMLb), measuring query wall time and output data volume from on-disk Arrow datasets.

## When NOT to use

- Input data is not in Parquet columnar format (e.g. proprietary UNIFI raw data not yet converted with arcMS, or mzML/HDF5 files) — convert first or use format-specific readers
- The goal is conversion-speed benchmarking of MSConvertGUI vs arcMS rather than on-disk query/read timing of already-converted files
- The query fully loads the file into memory via read_parquet(), since then no on-disk lazy evaluation is being measured

## Inputs

- HRMS data file in Apache Parquet format (e.g. cal1-full-long.parquet from the arcms-dataviz repository)
- R session with arrow, dplyr and DuckDB packages

## Outputs

- Measured in-memory dataset size (expected ~1.9 Gb for the example)
- TIC data frame collected in ~1s occupying ~64 Kb
- Binned rt/mz aggregation object of several Mb with Arrow vs DuckDB timing comparison
- Benchmark table of conversion time, file size, and read time per format (e.g. benchmark_table.csv)

## How to apply

Download or convert a sample (e.g. cal1-full-long.parquet from leesulab/arcms-dataviz) and first load it fully into R memory with arrow::read_parquet() to measure the in-memory footprint (~1.9 Gb for the example sample). Reopen the same file lazily with arrow::open_dataset() and time a dplyr query filtering mslevel == '1', grouping by rt and summing intensity to obtain the TIC; with collect() this should take around 1s and yield only ~64 Kb. Then time a binned aggregation (rt_binned = floor(rt/0.1)*0.1, mz_binned = floor(mz/1)*1) reducing data to several Mb, first via pure Arrow (~30s to 1 min) and then via to_duckdb() streaming, which runs in a few seconds — the rationale being that DuckDB pushes aggregation down more efficiently on larger reductions. Record wall times with a timer around each collect() and compare against the article's reported read times (Parquet 1-3 s vs mzML/DEIMoS 15-20 min, MZmine 1-2 min).

## Related tools

- **Arrow** (Used to open the Parquet file on disk with open_dataset() and run lazy dplyr filter/group_by/summarise queries (TIC, EIC, binned rt/mz) without loading all data into RAM) — https://github.com/leesulab/arcMS
- **arcMS** (R package that converts Waters UNIFI (HD)MSE data to the Parquet files that this timing measurement queries) — https://github.com/leesulab/arcMS
- **R** (Environment in which the timing benchmark dplyr/Arrow/DuckDB queries are executed and timed) — https://github.com/leesulab/arcMS

## Examples

```
data_arrow = arrow::open_dataset("cal1-full-long.parquet"); TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- The Arrow TIC query (filter mslevel == '1', group_by rt, sum intensity) completes in approximately 1s and the collected result is only ~64 Kb
- read_parquet()-loaded example data occupies ~1.9 Gb in memory
- The binned rt/mz aggregation reduces the data to several Mb; pure Arrow takes ~30s to 1 min while the to_duckdb() version runs in a few seconds
- Benchmark table contains the reported Parquet profile-mode file size range of ~200-400 MB for full IMS HDMSE data and Parquet read times of 1-3 s

## Limitations

- The pure Arrow binned rt/mz aggregation query takes ~30s to 1 min, much slower than DuckDB's few seconds, so Arrow-only timings can misrepresent achievable performance
- Reported timings and file sizes are for the environmental HRMS benchmark samples; values will vary with sample complexity and hardware
- No changelog is available for the arcMS repository, so version-dependent performance differences cannot be traced

## Evidence

- [methods] TIC via on-disk Arrow query timing and size: "TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()"
- [methods] TIC query time and output size: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] In-memory footprint of loaded example data: "the loaded data for this example sample occupies \~1.9 Gb in memory"
- [methods] Binned aggregation timing Arrow vs DuckDB: [evidence span withheld — anchor status: not_found]
- [readme] Parquet files are queried without full RAM load in the dataviz app: [evidence span withheld — anchor status: external_doc]
