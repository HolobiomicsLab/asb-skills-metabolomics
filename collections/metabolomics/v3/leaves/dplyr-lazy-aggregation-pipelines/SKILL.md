---
name: dplyr-lazy-aggregation-pipelines
description: Use when an HRMS Parquet file converted by arcMS is too large to load
  fully in RAM (e.g. ~1.9 Gb example) but chromatograms, spectra, or aggregated summaries
  must be extracted quickly.
when_to_use_negative:
- When the full dataset is actually needed in memory (use arrow::read_parquet instead)
- For profile-mode IMS HDMSE 2D binned aggregations where Arrow alone is slow (~30s-1
  min); prefer the DuckDB-backed to_duckdb() route
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Arrow
  role: Provides lazy, on-disk querying of the Parquet dataset so data is not fully
    loaded in RAM before aggregation
  repo: https://github.com/leesulab/arcMS
- name: dplyr
  role: Expresses the filter/group_by/summarise pipeline (e.g. filter(mslevel == "1"),
    group_by(rt), summarise(int = sum(intensity))) that Arrow executes lazily
  repo: https://github.com/leesulab/arcMS
- name: arcMS
  role: Converts Waters UNIFI (HD)MSE HRMS data to the Parquet files consumed by these
    Arrow/dplyr pipelines
  repo: https://github.com/leesulab/arcMS
- name: R
  role: Language in which the arcMS, arrow and dplyr pipeline is run
  repo: https://github.com/leesulab/arcMS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:cac4f1cacb068aaa32ff42968f86c54ce0b5d1e7e296fcfbaafa9ff07727af23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - R
  - arcMS
  - dplyr
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# dplyr-lazy-aggregation-pipelines

## Summary

Compose lazy dplyr pipelines over on-disk Apache Arrow datasets (arrow::open_dataset on Parquet files) so that HRMS filtering and aggregation (e.g. TIC computation) run on-disk and only the small aggregated result is collected into R memory.

## When to use

Use when an HRMS Parquet file converted by arcMS is too large to load fully in RAM (e.g. ~1.9 Gb example) but chromatograms, spectra, or aggregated summaries must be extracted quickly.

## When NOT to use

- When the full dataset is actually needed in memory (use arrow::read_parquet instead)
- For profile-mode IMS HDMSE 2D binned aggregations where Arrow alone is slow (~30s-1 min); prefer the DuckDB-backed to_duckdb() route

## Inputs

- Apache Parquet file of HRMS data converted with arcMS (e.g. cal1.parquet, ~1.9 Gb in memory)
- Arrow FileSystemDataset created with arrow::open_dataset()

## Outputs

- Small R data frame of aggregated values, e.g. a TIC with columns (rt, int) of ~64 Kb
- Optionally binned rt/mz aggregates or IMS traces / MS spectra depending on the filter and summarise steps

## How to apply

Open the Parquet file with arrow::open_dataset() to obtain a FileSystemDataset pointer without materializing the ~1.9 Gb of loaded data in memory. Build a dplyr pipeline directly on the Arrow dataset: filter(mslevel == "1") to keep only low collision energy (MS1) scans, group_by(rt), and summarise(int = sum(intensity)) to compute the total ion chromatogram. Call collect() only at the end so just the aggregated result is brought into memory. The paper reports this took around 1s and the resulting data is only 64 Kb. For heavier binned 2D aggregations the same pipeline can be routed to DuckDB via to_duckdb(), which runs in a few seconds versus ~30s to 1 min for the pure Arrow query.

## Related tools

- **Arrow** (Provides lazy, on-disk querying of the Parquet dataset so data is not fully loaded in RAM before aggregation) — https://github.com/leesulab/arcMS
- **dplyr** (Expresses the filter/group_by/summarise pipeline (e.g. filter(mslevel == "1"), group_by(rt), summarise(int = sum(intensity))) that Arrow executes lazily) — https://github.com/leesulab/arcMS
- **arcMS** (Converts Waters UNIFI (HD)MSE HRMS data to the Parquet files consumed by these Arrow/dplyr pipelines) — https://github.com/leesulab/arcMS
- **R** (Language in which the arcMS, arrow and dplyr pipeline is run) — https://github.com/leesulab/arcMS

## Examples

```
TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- Output contains exactly two columns (rt, int) for the TIC query
- Resulting collected data size is on the order of tens of Kb (~64 Kb reported for the example)
- Intensity values are positive sums of low-energy (mslevel == "1") scan intensities across the full retention-time range
- Query completes in about 1s for the TIC (per the paper)
- The ~1.9 Gb Parquet dataset was never fully loaded into R memory

## Limitations

- Pure Arrow binned rt/mz aggregation queries can take ~30s to 1 min; DuckDB (to_duckdb()) runs the same query in a few seconds
- HDF5 output files are fast to access but larger than Parquet
- Converting a full profile-mode file containing IMS HDMS'E data yields Parquet files of ~200-400 MB

## Evidence

- [methods] TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect(): "TIC = data_arrow |>
  filter(mslevel == "1") |>
  group_by(rt) |> 
  summarise(int = sum(intensity)) |> 
  collect()"
- [methods] This took around 1s and the resulting data is only 64 Kb!: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] the loaded data for this example sample occupies ~1.9 Gb in memory: "the loaded data for this example sample occupies \~1.9 Gb in memory"
- [readme] Data is not fully loaded in RAM but chromatograms and MS spectra are quickly retrieved from the parquet file with the Apache Arrow library.: "Data is not fully loaded in RAM but chromatograms and MS spectra are
quickly retrieved from the parquet file with the [Apache Arrow
library](https://arrow.apache.org/)."
- [methods] The resulting object is now just several Mbs, but the query takes ~30s to 1 min ... This now runs in a few seconds: [evidence span withheld — anchor status: not_found]
