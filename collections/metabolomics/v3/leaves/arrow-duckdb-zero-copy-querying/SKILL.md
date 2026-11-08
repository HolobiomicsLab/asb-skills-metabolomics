---
name: arrow-duckdb-zero-copy-querying
description: Use when a Parquet-stored HRMS dataset is too large for RAM and you need
  filtered aggregates such as an EIC for a specific m/z window (e.g. 216.0016 < mz
  < 216.2016 around atrazine) or a TIC, computed with SQL-grade speed.
when_to_use_negative:
- The data is already small enough to fit in memory — a plain arrow::read_parquet()
  load followed by dplyr is simpler
- The input is raw vendor/UNIFI data or mzML rather than an arcMS-converted Parquet
  file — convert it first with arcMS (convert_one_sample_data / convert_mzml_to_parquet)
- You need output in HDF5 instead of Parquet — this skill targets the Parquet/Arrow
  path; HDF5 files are read with rhdf5 datasets (ms1/ms2) instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3077
tools:
- name: Arrow
  role: Opens the Parquet dataset on-disk with arrow::open_dataset() so the file is
    not loaded into RAM, then streams it to DuckDB
  repo: https://arrow.apache.org/
- name: DuckDB
  role: Receives the zero-copy Arrow stream via to_duckdb() and executes fast SQL
    filtering/aggregation (e.g. binned rt/mz queries in a few seconds vs ~30s-1min
    with Arrow alone)
- name: arcMS
  role: Produces the Parquet HRMS input files from Waters UNIFI (HD)MSE data; the
    query skill operates on these Parquet outputs
  repo: https://github.com/leesulab/arcMS
- name: dplyr
  role: Expresses the filter/group_by/summarise pipeline (e.g. summarise(intensity
    = sum(intensity))) over the DuckDB-backed lazy table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:43408f740534f3bfa19b42c66b91aa7decdd8bedaf2245be8391ffb6f914810f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Arrow
  - DuckDB
  - R
  - arcMS
  - dplyr
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# arrow-duckdb-zero-copy-querying

## Summary

Query large Parquet-backed HRMS datasets (e.g. ~200-400 MB profile-mode HDMSE files from arcMS) on-disk by streaming an Arrow dataset into DuckDB with to_duckdb(), enabling fast SQL/dplyr filtering and aggregation (TIC, EIC, spectra) without loading the file into RAM or duplicating data.

## When to use

Use when a Parquet-stored HRMS dataset is too large for RAM and you need filtered aggregates such as an EIC for a specific m/z window (e.g. 216.0016 < mz < 216.2016 around atrazine) or a TIC, computed with SQL-grade speed.

## When NOT to use

- The data is already small enough to fit in memory — a plain arrow::read_parquet() load followed by dplyr is simpler
- The input is raw vendor/UNIFI data or mzML rather than an arcMS-converted Parquet file — convert it first with arcMS (convert_one_sample_data / convert_mzml_to_parquet)
- You need output in HDF5 instead of Parquet — this skill targets the Parquet/Arrow path; HDF5 files are read with rhdf5 datasets (ms1/ms2) instead

## Inputs

- Apache Parquet file of arcMS-converted HRMS data (e.g. cal1-full-long.parquet, ~200-400 MB profile-mode HDMSE with IMS)
- Columns: mz, intensity, rt, scanid, mslevel, bin (drift-time bin index)
- Target analyte m/z and window half-width (e.g. 216.1016 +/- 0.1)

## Outputs

- Small aggregated table (e.g. EIC with rt and summed intensity columns, or TIC with rt and int columns)
- CSV export such as eic_atrazine.csv

## How to apply

Open the on-disk Parquet file with arrow::open_dataset() so the multi-hundred-MB profile-mode file is never fully loaded into memory. Stream the Arrow dataset into DuckDB via to_duckdb() for zero-copy access to a SQL/dplyr query interface. Filter to low-collision-energy data (mslevel == 1) and window the m/z values (e.g. +/-0.1 around the target, such as 216.0016 < mz < 216.2016 for atrazine at m/z 216.1016), then group_by(rt, scanid) and summarise(intensity = sum(intensity)). Collect() only the small resulting table, arrange by retention time, and save (e.g. eic_atrazine.csv). For heavier binned rt/mz aggregations, prefer the DuckDB path: the pure Arrow query takes ~30s to 1 min while DuckDB runs in a few seconds; a TIC equivalent produced only ~64 Kb of output.

## Related tools

- **Arrow** (Opens the Parquet dataset on-disk with arrow::open_dataset() so the file is not loaded into RAM, then streams it to DuckDB) — https://arrow.apache.org/
- **DuckDB** (Receives the zero-copy Arrow stream via to_duckdb() and executes fast SQL filtering/aggregation (e.g. binned rt/mz queries in a few seconds vs ~30s-1min with Arrow alone))
- **arcMS** (Produces the Parquet HRMS input files from Waters UNIFI (HD)MSE data; the query skill operates on these Parquet outputs) — https://github.com/leesulab/arcMS
- **dplyr** (Expresses the filter/group_by/summarise pipeline (e.g. summarise(intensity = sum(intensity))) over the DuckDB-backed lazy table)

## Examples

```
data_arrow <- arrow::open_dataset("cal1.parquet"); eic <- data_arrow |> to_duckdb() |> filter(mslevel == "1", mz > 216.0016 & mz < 216.2016) |> group_by(rt, scanid) |> summarise(intensity = sum(intensity)) |> collect()
```

## Evaluation signals

- Output CSV (e.g. eic_atrazine.csv) exists and contains exactly the rt and intensity columns
- All summed intensity values are positive and the retention times are ordered increasingly
- Result table is small ( Kb-to-Mb scale), consistent with on-disk aggregation — the TIC equivalent produced only ~64 Kb
- R memory footprint stays far below the ~1.9 Gb the loaded example dataset would occupy, confirming no full in-memory load
- DuckDB-backed binned rt/mz aggregation completes in a few seconds rather than the ~30s-1min of the pure Arrow query

## Limitations

- Pure Arrow aggregation (without DuckDB) is slow for binned rt/mz queries: ~30s to 1 min
- The loaded example dataset occupies ~1.9 Gb in memory if read fully with read_parquet, so full in-memory loading defeats the purpose
- The input must be an arcMS-converted Parquet table with mslevel/mz/rt/scanid/bin columns; HDF5 outputs and raw vendor formats are not directly queryable this way

## Evidence

- [methods] Advanced filtering of Parquet-backed HRMS data is achieved by querying the Arrow dataset directly with the DuckDB database system through an SQL interface, without any data duplication.: [evidence span withheld — anchor status: not_found]
- [methods] DuckDB outperforms pure Arrow on binned aggregations.: [evidence span withheld — anchor status: not_found]
- [methods] TIC aggregation output size shows on-disk aggregation works.: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] EIC m/z window filter around atrazine.: "filter(mz > 216.0016 & mz < 216.2016) |> 
  group_by(rt, scanid) |>
  summarise(intensity = sum(intensity))"
- [readme] README confirms on-disk Arrow retrieval of chromatograms from Parquet.: "Data is not fully loaded in RAM but chromatograms and MS spectra are quickly retrieved from the parquet file with the [Apache Arrow
library](https://arrow.apache.org/)"
- [methods] Example dataset memory footprint justifies on-disk querying.: "the loaded data for this example sample occupies \~1.9 Gb in memory"
