---
name: arrow-on-disk-parquet-querying
description: Use when an HRMS Parquet file (e.g., arcMS-converted Waters UNIFI data)
  is too large to load into memory and you need derived quantities like TIC, BPI,
  EIC or MS spectra.
when_to_use_negative:
- The analysis needs the full peak table in memory (e.g., interactive row-level inspection
  or complex modeling on all peaks) — use arrow::read_parquet() instead
- The input is an HDF5 (.h5) file from arcMS rather than Parquet — use rhdf5::h5read()
  on the ms1/ms2 datasets instead
- The query is a heavy binned rt/mz or IMS aggregation where pure Arrow takes ~30s
  to 1 min — route through to_duckdb() instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Arrow
  role: Provides open_dataset() for on-disk lazy querying of the Parquet file and
    vectorized filter/group_by/summarise aggregation without loading full data into
    RAM
  repo: https://arrow.apache.org/
- name: dplyr
  role: Grammar used to express the lazy pipeline (filter, group_by, summarise, collect)
    on the Arrow dataset
- name: arcMS
  role: Converts Waters UNIFI (HD)MS^E data into the Parquet columnar format that
    this querying skill operates on
  repo: https://github.com/leesulab/arcMS
- name: DuckDB
  role: Optional backend via to_duckdb() for binned rt/mz and IMS-trace aggregations
    that run in a few seconds instead of ~30s to 1 min with pure Arrow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:2d3b6e769c6c2abbd0b45463c9ee2fd9acc4c1a6d10a3e0d22079a24bc1bba9a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Arrow
  - DuckDB
  - R
  - arcMS
  - dplyr
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# arrow-on-disk-parquet-querying

## Summary

Compute chromatograms or spectra (e.g., a TIC) from a large HRMS Parquet file using an on-disk Apache Arrow/dplyr pipeline, so that only the small aggregated result is loaded into RAM instead of the full ~1.9 Gb dataset.

## When to use

Use when an HRMS Parquet file (e.g., arcMS-converted Waters UNIFI data) is too large to load into memory and you need derived quantities like TIC, BPI, EIC or MS spectra.

## When NOT to use

- The analysis needs the full peak table in memory (e.g., interactive row-level inspection or complex modeling on all peaks) — use arrow::read_parquet() instead
- The input is an HDF5 (.h5) file from arcMS rather than Parquet — use rhdf5::h5read() on the ms1/ms2 datasets instead
- The query is a heavy binned rt/mz or IMS aggregation where pure Arrow takes ~30s to 1 min — route through to_duckdb() instead

## Inputs

- Apache Parquet file of HRMS peak-list data converted with arcMS (columns such as rt, mz, intensity, mslevel, scanid, bin), e.g. cal1-full-long.parquet (~1.9 Gb loaded in memory)
- An arrow FileSystemDataset created via arrow::open_dataset() from the Parquet file

## Outputs

- A small R data frame (~64 Kb) with columns rt and int representing the total ion chromatogram
- Optionally EICs (m/z window filtered), binned rt/mz surfaces, IMS traces, or MS spectra, as aggregated data frames

## How to apply

Open the Parquet file lazily with arrow::open_dataset() to create a FileSystemDataset pointer without materializing the ~1.9 Gb of data in RAM. Build a dplyr pipeline directly on the Arrow dataset: filter on mslevel == "1" to keep only low collision energy (MS1) scans, group by rt (and optionally scanid or mz depending on the target chromatogram/spectrum), then summarise with sum(intensity). Call collect() only at the end so that only the aggregated result is pulled into R as a data frame. For heavier aggregation queries (e.g., binned rt/mz surfaces or IMS traces), pipe the Arrow dataset through to_duckdb() to run in a few seconds instead of ~30s to 1 min with pure Arrow. Rationale: Parquet columnar storage plus Arrow predicate/aggregate pushdown lets disk-resident data be filtered and summed with only tens of Kb of output collected.

## Related tools

- **Arrow** (Provides open_dataset() for on-disk lazy querying of the Parquet file and vectorized filter/group_by/summarise aggregation without loading full data into RAM) — https://arrow.apache.org/
- **dplyr** (Grammar used to express the lazy pipeline (filter, group_by, summarise, collect) on the Arrow dataset)
- **arcMS** (Converts Waters UNIFI (HD)MS^E data into the Parquet columnar format that this querying skill operates on) — https://github.com/leesulab/arcMS
- **DuckDB** (Optional backend via to_duckdb() for binned rt/mz and IMS-trace aggregations that run in a few seconds instead of ~30s to 1 min with pure Arrow)

## Examples

```
data_arrow = arrow::open_dataset("cal1.parquet"); TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- Output data frame has exactly two columns (rt, int) for the TIC
- Aggregated intensity values are positive sums of low-energy (mslevel == "1") scan intensities spanning the full retention-time range
- Resulting table size is on the order of tens of Kb (~64 Kb), not Gb, confirming only the aggregate was collected
- Query completes in around 1s for the TIC case as reported for the cal1.parquet example
- The Arrow dataset object is a FileSystemDataset pointer, not an in-memory table

## Limitations

- Pure Arrow binned rt/mz aggregation queries can take ~30s to 1 min; DuckDB (to_duckdb()) runs the same query in a few seconds
- Only aggregated results (chromatograms/spectra) are collected; full-file inspection requires read_parquet() and ~1.9 Gb of RAM for the example sample
- Requires the input to be arcMS-converted Parquet with expected columns (mslevel, rt, intensity); HDF5 outputs use a different access path (rhdf5)

## Evidence

- [methods] TIC via open_dataset + dplyr aggregation: "TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()"
- [methods] On-disk query speed and size: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] In-memory footprint of loaded data: "the loaded data for this example sample occupies ~1.9 Gb in memory"
- [methods] DuckDB speedup for binned queries: "The resulting object is now just several Mbs, but the query takes ~30s to 1 min"
- [readme] Dataviz app on-disk access: [evidence span withheld — anchor status: not_found]
