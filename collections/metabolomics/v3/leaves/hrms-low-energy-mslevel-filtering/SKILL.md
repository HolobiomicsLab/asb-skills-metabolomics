---
name: hrms-low-energy-mslevel-filtering
description: Use when you have HRMS data in Apache Parquet format (from arcMS/HDMSE
  conversion) and need to extract an EIC, TIC or MS spectrum for a specific m/z window
  or scan subset without loading the full file into memory.
when_to_use_negative:
- Input is an mzML, UNIFI-native or HDF5 file that has not yet been converted to Parquet
  (use arcMS conversion first, e.g. convert_one_sample_data())
- You need high-collision-energy (MS2 / high-energy) fragment data rather than MS1-level
  traces — this skill explicitly filters to mslevel == "1"
- The whole dataset already fits in RAM and a simple in-memory dplyr or pure Arrow
  query is fast enough (the article notes pure Arrow binned queries can take ~30s–1
  min, so a small in-memory file does not need DuckDB)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: arcMS
  role: R package that converts Waters UNIFI (HD)MSE data to Parquet/HDF5, producing
    the columnar input queried by this skill
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Opens the Parquet dataset on disk (arrow::open_dataset()) and supports lazy
    filtering/aggregation without loading data into RAM
- name: DuckDB
  role: Receives the Arrow dataset via zero-copy to_duckdb() and executes the SQL-backed
    mslevel/mz window filtering and aggregation in a few seconds
- name: dplyr
  role: Provides the filter(mslevel == "1"), filter(mz > ...), group_by(rt, scanid)
    and summarise(intensity = sum(intensity)) verb syntax used over the DuckDB-backed
    table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:7acb61e9ec451cf53f575e709b1cb59f078d1f0a1bf22499925c1e9aac9914ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - DuckDB
  - R
  - arcMS
  - dplyr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# hrms-low-energy-mslevel-filtering

## Summary

Extracts chromatograms or spectra from Parquet-stored HRMS data by filtering to low-collision-energy (mslevel == 1) scans and windowing m/z values (e.g. 216.0016–216.2016 for atrazine), querying the on-disk Arrow dataset through DuckDB without loading the ~200–400 MB profile-mode file into RAM.

## When to use

Use when you have HRMS data in Apache Parquet format (from arcMS/HDMSE conversion) and need to extract an EIC, TIC or MS spectrum for a specific m/z window or scan subset without loading the full file into memory.

## When NOT to use

- Input is an mzML, UNIFI-native or HDF5 file that has not yet been converted to Parquet (use arcMS conversion first, e.g. convert_one_sample_data())
- You need high-collision-energy (MS2 / high-energy) fragment data rather than MS1-level traces — this skill explicitly filters to mslevel == "1"
- The whole dataset already fits in RAM and a simple in-memory dplyr or pure Arrow query is fast enough (the article notes pure Arrow binned queries can take ~30s–1 min, so a small in-memory file does not need DuckDB)

## Inputs

- Apache Parquet file of HRMS profile-mode data converted with arcMS (e.g. cal1-full-long.parquet, ~200-400 MB, containing mslevel, mz, intensity, rt, scanid columns)
- Target m/z value and window half-width (e.g. 216.1016 ± 0.1 for atrazine)

## Outputs

- Small aggregated data frame / CSV table with rt and summed intensity columns (e.g. eic_atrazine.csv) representing the extracted-ion chromatogram

## How to apply

Open the on-disk Parquet file with arrow::open_dataset() so the ~200–400 MB profile-mode HDMS^E file is never fully loaded into RAM, then stream it into DuckDB with to_duckdb() for zero-copy SQL querying. Filter with mslevel == "1" to retain only low-collision-energy (MS1) scans, since Parquet files from arcMS contain both low- and high-energy spectra. For an EIC, further window the m/z values (e.g. filter(mz > 216.0016 & mz < 216.2016), i.e. ±0.1 around the atrazine m/z of 216.1016), then group_by(rt, scanid) and summarise(intensity = sum(intensity)) in dplyr syntax. Call collect() only at the end so only the small aggregated table (e.g. a ~64 Kb TIC) is materialized in memory; arrange by increasing retention time and save the result (e.g. eic_atrazine.csv).

## Related tools

- **arcMS** (R package that converts Waters UNIFI (HD)MSE data to Parquet/HDF5, producing the columnar input queried by this skill) — https://github.com/leesulab/arcMS
- **Arrow** (Opens the Parquet dataset on disk (arrow::open_dataset()) and supports lazy filtering/aggregation without loading data into RAM)
- **DuckDB** (Receives the Arrow dataset via zero-copy to_duckdb() and executes the SQL-backed mslevel/mz window filtering and aggregation in a few seconds)
- **dplyr** (Provides the filter(mslevel == "1"), filter(mz > ...), group_by(rt, scanid) and summarise(intensity = sum(intensity)) verb syntax used over the DuckDB-backed table)

## Examples

```
eic = data_arrow |> to_duckdb() |> filter(mslevel == "1") |> filter(mz > 216.0016 & mz < 216.2016) |> group_by(rt, scanid) |> summarise(intensity = sum(intensity)) |> collect()
```

## Evaluation signals

- The output CSV (e.g. eic_atrazine.csv) exists and contains exactly the rt and intensity columns
- All summed intensity values are positive and the table is small (consistent with on-disk aggregation, e.g. the equivalent TIC query returned only ~64 Kb)
- All rows correspond to mslevel == 1 (low collision energy) and m/z values fall inside the specified window (216.0016 < mz < 216.2016)
- Rows are ordered by increasing retention time
- The full Parquet file was never loaded into RAM (the loaded example data would otherwise occupy ~1.9 Gb in memory)

## Limitations

- Pure Arrow binned aggregation queries (without DuckDB) can take ~30s to 1 min, whereas DuckDB runs in a few seconds
- Only works on Parquet files produced with arcMS column schema (mslevel, mz, intensity, rt, scanid); Parquet files contain both low and high energy spectra, so mslevel filtering is mandatory for MS1 traces
- No changelog available for the arcMS repository, so version behavior of the package is not documented

## Evidence

- [methods] mslevel filter on Parquet-backed data: "TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()"
- [methods] EIC m/z window around atrazine: "filter(mz > 216.0016 & mz < 216.2016) |> group_by(rt, scanid) |> summarise(intensity = sum(intensity))"
- [methods] on-disk aggregation output size: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] Parquet file size and scope: "A full data file in profile mode containing IMS, and HDMS<sup>E</sup> data (MS and MS/MS), is around 200-400 Mo."
- [readme] Parquet files contain both energy levels: [evidence span withheld — anchor status: external_doc]
