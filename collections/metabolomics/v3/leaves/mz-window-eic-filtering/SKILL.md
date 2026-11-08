---
name: mz-window-eic-filtering
description: Use when you need an EIC (or targeted ion trace) for a specific m/z window
  from a large Parquet-converted HRMS file and want on-disk, zero-copy filtering rather
  than full in-memory loading.
when_to_use_negative:
- Input data has not been converted to Parquet/HDF5 columnar format by arcMS (e.g.
  raw Waters UNIFI or mzML files) — convert first with convert_one_sample_data()
- You need a full total ion chromatogram or whole-file manipulation that requires
  all scans in memory, not a narrow m/z window
- Data is already a small in-memory table read with arrow::read_parquet — the Arrow/DuckDB
  lazy query machinery adds no benefit
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: arcMS
  role: R package that converts Waters UNIFI (HD)MSE data to Parquet so the EIC query
    can run on columnar on-disk data
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: arrow::open_dataset() opens the Parquet file lazily on disk without loading
    it into RAM
- name: DuckDB
  role: Receives the Arrow dataset via to_duckdb() zero-copy streaming to execute
    the m/z-window filter and intensity aggregation in seconds
- name: dplyr
  role: Provides the filter/group_by/summarise/collect syntax used to express the
    EIC query
- name: Shiny
  role: Companion arcms-dataviz app demonstrates visualization of TIC, BPI or EIC
    chromatograms from the same Parquet data
  repo: https://github.com/leesulab/arcms-dataviz
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:d303f79d48c1d7d808049541b687a551206f2caf618b5f4110f7511b3af11263
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - DuckDB
  - R
  - Shiny
  - arcMS
  - dplyr
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mz-window-eic-filtering

## Summary

Extract an extracted-ion chromatogram (EIC) for a narrow m/z window from a Parquet-stored HRMS file by streaming an Arrow dataset into DuckDB and aggregating intensity by retention time, without loading the ~200-400 MB profile-mode file into RAM.

## When to use

Use when you need an EIC (or targeted ion trace) for a specific m/z window from a large Parquet-converted HRMS file and want on-disk, zero-copy filtering rather than full in-memory loading.

## When NOT to use

- Input data has not been converted to Parquet/HDF5 columnar format by arcMS (e.g. raw Waters UNIFI or mzML files) — convert first with convert_one_sample_data()
- You need a full total ion chromatogram or whole-file manipulation that requires all scans in memory, not a narrow m/z window
- Data is already a small in-memory table read with arrow::read_parquet — the Arrow/DuckDB lazy query machinery adds no benefit

## Inputs

- Apache Parquet file of arcMS-converted HRMS profile-mode data (e.g. cal1-full-long.parquet, ~200-400 MB, with columns mz, intensity, rt, scanid, mslevel, bin)
- Target analyte m/z and desired window half-width (e.g. +/-0.1 around m/z 216.1016 for atrazine)

## Outputs

- Small EIC table with rt and summed intensity columns (e.g. eic_atrazine.csv), kilobyte-scale rather than the full file
- Optionally a chromatogram plot of intensity vs retention time

## How to apply

Open the on-disk Parquet file with arrow::open_dataset() so the ~200-400 MB profile-mode HDMSE file is never fully loaded into RAM, then stream it into DuckDB with to_duckdb() for a zero-copy SQL query interface. Filter to low-collision-energy data with mslevel == '1' and window the m/z values to +/-0.1 around the target ion (e.g. 216.0016 < mz < 216.2016 for atrazine at m/z 216.1016). Group by rt (and scanid) and summarise intensity = sum(intensity) in dplyr syntax, then collect() only the small aggregated table, arrange by increasing retention time, and write it out (e.g. eic_atrazine.csv). The rationale is that DuckDB executes complex filters/aggregations in a few seconds where pure Arrow binned queries can take ~30 s to 1 min, and only kilobyte-scale results are materialized (the TIC equivalent yielded only ~64 Kb).

## Related tools

- **arcMS** (R package that converts Waters UNIFI (HD)MSE data to Parquet so the EIC query can run on columnar on-disk data) — https://github.com/leesulab/arcMS
- **Arrow** (arrow::open_dataset() opens the Parquet file lazily on disk without loading it into RAM)
- **DuckDB** (Receives the Arrow dataset via to_duckdb() zero-copy streaming to execute the m/z-window filter and intensity aggregation in seconds)
- **dplyr** (Provides the filter/group_by/summarise/collect syntax used to express the EIC query)
- **Shiny** (Companion arcms-dataviz app demonstrates visualization of TIC, BPI or EIC chromatograms from the same Parquet data) — https://github.com/leesulab/arcms-dataviz

## Examples

```
eic_atrazine = data_arrow |> to_duckdb() |> filter(mslevel == "1") |> filter(mz > 216.0016 & mz < 216.2016) |> group_by(rt, scanid) |> summarise(intensity = sum(intensity)) |> collect()
```

## Evaluation signals

- Output CSV (e.g. eic_atrazine.csv) exists and contains exactly rt and intensity columns
- All summed intensity values are positive
- Table is kilobyte-scale (small, consistent with on-disk aggregation; the analogous TIC query produced only ~64 Kb)
- Rows are arranged by increasing retention time and every m/z value used lies within the requested window (e.g. 216.0016 < mz < 216.2016)
- Query completed without loading the full ~1.9 Gb in-memory footprint of the sample (loaded data occupies ~1.9 Gb in memory)

## Limitations

- Pure Arrow binned rt/mz aggregation queries can take ~30 s to 1 min; DuckDB is needed to bring this down to a few seconds
- Applies to Parquet-backed arcMS output (MSE/HDMSE from Waters UNIFI or converted mzML); HDF5 files are read with rhdf5 and structured as ms1/ms2 datasets instead
- Example files like cal1-full-long.parquet occupy ~1.9 Gb when fully loaded, so naive in-memory approaches defeat the purpose

## Evidence

- [methods] Zero-copy Arrow-to-DuckDB streaming rationale: "rtmzbinned = data_arrow |>
    to_duckdb() |>
    filter(mslevel == "1")"
- [methods] EIC m/z window around atrazine with rt/scanid aggregation: "filter(mz > 216.0016 & mz < 216.2016) |> 
  group_by(rt, scanid) |>
  summarise(intensity = sum(intensity))"
- [methods] DuckDB speed advantage over pure Arrow binned queries: [evidence span withheld — anchor status: not_found]
- [methods] On-disk aggregation produces tiny results: "This took around 1s and the resulting data is only 64 Kb!"
- [readme] README: visualization of chromatograms without full RAM load: "Data is not fully loaded in RAM but chromatograms and MS spectra are
quickly retrieved from the parquet file with the [Apache Arrow
library]"
