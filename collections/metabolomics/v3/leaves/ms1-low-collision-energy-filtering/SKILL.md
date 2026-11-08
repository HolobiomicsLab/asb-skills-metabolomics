---
name: ms1-low-collision-energy-filtering
description: Use when computing TIC, EIC, or MS1 spectra from arcMS-converted Parquet
  files containing both low and high energy (HDMSE) scans and only precursor (MS1)
  signal should be aggregated.
when_to_use_negative:
- The analysis targets fragment (MS2 / high collision energy) signal, e.g. MS/MS spectra
  — filtering mslevel == "1" would discard exactly those scans.
- 'The input is HDF5 output rather than Parquet: HDF5 files already separate low energy
  into the ''ms1'' dataset and high energy into ''ms2'', so no mslevel filter is needed.'
- The full dataset is small enough to load with arrow::read_parquet() and needs non-filterable
  whole-table operations on all scans regardless of collision energy.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: arcMS
  role: R package that converts Waters UNIFI (HD)MSE data (or mzML) to Parquet with
    an mslevel column enabling MS1 filtering
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Provides open_dataset() on-disk querying and dplyr-compatible filter(mslevel
    == "1") on Parquet without loading ~1.9 Gb into RAM
- name: dplyr
  role: Pipeline verbs (filter, group_by, summarise, collect) used to express the
    MS1 filter and downstream aggregation
- name: DuckDB
  role: Alternative backend via to_duckdb() that runs the MS1-filtered binned aggregation
    query in a few seconds instead of ~30s-1min
- name: Shiny (arcms-dataviz)
  role: Companion app demonstrating the same on-disk MS1-filtered retrieval of chromatograms
    from Parquet
  repo: https://github.com/leesulab/arcms-dataviz
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:a4b36dbcc014437039b3cb16de9e1ce0270de48aaf869ce558c5c21a2b9a709c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - DuckDB
  - R
  - Shiny (arcms-dataviz)
  - arcMS
  - dplyr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# ms1-low-collision-energy-filtering

## Summary

Filters HRMS Parquet data (converted with arcMS) to low collision energy scans via `filter(mslevel == "1")` in an Arrow/dplyr pipeline, isolating MS1 precursor-level signal before chromatogram or spectrum aggregation.

## When to use

Use when computing TIC, EIC, or MS1 spectra from arcMS-converted Parquet files containing both low and high energy (HDMSE) scans and only precursor (MS1) signal should be aggregated.

## When NOT to use

- The analysis targets fragment (MS2 / high collision energy) signal, e.g. MS/MS spectra — filtering mslevel == "1" would discard exactly those scans.
- The input is HDF5 output rather than Parquet: HDF5 files already separate low energy into the 'ms1' dataset and high energy into 'ms2', so no mslevel filter is needed.
- The full dataset is small enough to load with arrow::read_parquet() and needs non-filterable whole-table operations on all scans regardless of collision energy.

## Inputs

- arcMS-converted Apache Parquet file containing low and high energy spectra (HDMSE/MSE), e.g. cal1-full-long.parquet
- Arrow FileSystemDataset pointer created via arrow::open_dataset()

## Outputs

- R data frame of aggregated MS1-level results (e.g. rt/int TIC columns, ~64 Kb for the example TIC)
- MS1-only chromatograms or spectra for downstream plotting

## How to apply

Open the Parquet file on-disk with arrow::open_dataset() (e.g. the ~1.9 Gb in-memory cal1.parquet example from arcms-dataviz) so data is not fully loaded into RAM. Apply filter(mslevel == "1") as the first dplyr verb on the Arrow dataset to keep only low collision energy (MS1) scans. Then aggregate downstream: group_by(rt) with summarise(int = sum(intensity)) for a TIC, group_by(rt, scanid) after an m/z window filter (e.g. mz > 216.0016 & mz < 216.2016 for atrazine) for an EIC, or group_by(mz) after scanid/bin filters for an MS spectrum. Call collect() only on the aggregated result — the TIC example produced ~64 Kb in ~1s versus the full ~1.9 Gb dataset. For heavier binned aggregations, pipe to_duckdb() instead of pure Arrow to run the query in a few seconds rather than ~30s-1min.

## Related tools

- **arcMS** (R package that converts Waters UNIFI (HD)MSE data (or mzML) to Parquet with an mslevel column enabling MS1 filtering) — https://github.com/leesulab/arcMS
- **Arrow** (Provides open_dataset() on-disk querying and dplyr-compatible filter(mslevel == "1") on Parquet without loading ~1.9 Gb into RAM)
- **dplyr** (Pipeline verbs (filter, group_by, summarise, collect) used to express the MS1 filter and downstream aggregation)
- **DuckDB** (Alternative backend via to_duckdb() that runs the MS1-filtered binned aggregation query in a few seconds instead of ~30s-1min)
- **Shiny (arcms-dataviz)** (Companion app demonstrating the same on-disk MS1-filtered retrieval of chromatograms from Parquet) — https://github.com/leesulab/arcms-dataviz

## Examples

```
TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- Output contains exactly the expected aggregation columns (e.g. rt, int for a TIC) and no raw scan rows
- All summed intensity values are positive and span the full retention-time range of the MS1 scans
- Collected result size is on the order of tens of Kb (paper: ~64 Kb) rather than the ~1.9 Gb full dataset
- Query completes in ~1s for the pure Arrow TIC path; binned rt/mz aggregation with to_duckdb() should run in a few seconds
- Spot-check that only mslevel == '1' rows contributed by comparing against an equivalent in-memory dplyr filter

## Limitations

- The mslevel value is stored as a character string, so the filter must use filter(mslevel == "1") not numeric 1.
- Pure Arrow binned rt/mz aggregation is slow (~30s to 1 min); DuckDB (to_duckdb()) is needed for second-scale performance.
- Filtering works only on arcMS Parquet layout with an mslevel column; HDF5 output instead stores low/high energy in separate 'ms1'/'ms2' datasets.

## Evidence

- [methods] TIC aggregation via on-disk Arrow query produces only ~64 Kb of data: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] MS1 filter in the TIC pipeline: "TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()"
- [methods] EIC m/z window after MS1 filtering: "filter(mz > 216.0016 & mz < 216.2016) |> group_by(rt, scanid) |> summarise(intensity = sum(intensity))"
- [readme] Parquet contains both energy levels: [evidence span withheld — anchor status: external_doc]
- [readme] On-disk querying without full RAM load: [evidence span withheld — anchor status: not_found]
