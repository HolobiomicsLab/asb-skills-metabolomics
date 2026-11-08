---
name: tic-aggregation-from-arrow-datasets
description: Use when HRMS data has been converted to Parquet with arcMS and you need
  a TIC (or similar chromatogram) without loading the full profile-mode file into
  RAM, e.g. for Waters UNIFI (HD)MSE acquisitions with ion mobility.
when_to_use_negative:
- The data is still in Waters UNIFI or proprietary format and has not yet been converted
  to Parquet/HDF5 with arcMS — convert it first.
- You need the full point-level dataset in memory for custom modeling, not aggregated
  chromatograms/spectra — read_parquet() may be more appropriate despite the ~1.9
  Gb footprint.
- The file is HDF5 output from arcMS (ms1/ms2 datasets read via rhdf5), where the
  Arrow dataset workflow does not apply.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: arcMS
  role: Converts Waters UNIFI (HD)MSE HRMS data into the Apache Parquet columnar files
    that this TIC query consumes
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Provides open_dataset() lazy on-disk querying so the TIC aggregation filters
    mslevel == 1 and sums intensity by rt without loading ~1.9 Gb into RAM
  repo: https://github.com/leesulab/arcMS
- name: Shiny
  role: Companion arcms-dataviz app retrieves TIC and spectra from Parquet files with
    Arrow for browser-based visualization
  repo: https://github.com/leesulab/arcms-dataviz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:9ae29b671956f48e6a465114cc988d8fcec1daff262697364e89135517e94135
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - R
  - Shiny
  - arcMS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# tic-aggregation-from-arrow-datasets

## Summary

Compute total ion chromatograms (TIC) from arcMS-converted Parquet HRMS data using Apache Arrow on-disk queries (filter mslevel == 1, group by rt, sum intensity), avoiding full in-memory loading. This exploits Parquet columnar storage so a ~1.9 Gb dataset yields only ~64 Kb of aggregated chromatogram data.

## When to use

Use when HRMS data has been converted to Parquet with arcMS and you need a TIC (or similar chromatogram) without loading the full profile-mode file into RAM, e.g. for Waters UNIFI (HD)MSE acquisitions with ion mobility.

## When NOT to use

- The data is still in Waters UNIFI or proprietary format and has not yet been converted to Parquet/HDF5 with arcMS — convert it first.
- You need the full point-level dataset in memory for custom modeling, not aggregated chromatograms/spectra — read_parquet() may be more appropriate despite the ~1.9 Gb footprint.
- The file is HDF5 output from arcMS (ms1/ms2 datasets read via rhdf5), where the Arrow dataset workflow does not apply.

## Inputs

- arcMS-converted Apache Parquet file of (HD)MSE HRMS data (e.g. cal1-full-long.parquet from leesulab/arcms-dataviz)
- arrow Dataset object created with open_dataset() on the Parquet file

## Outputs

- TIC as an R data frame with columns rt and summed intensity (~64 Kb for the example sample, collected in ~1s)
- Optionally BPI/EIC chromatograms or binned rt/mz heatmaps (several Mb) using the same Arrow/DuckDB query pattern

## How to apply

Open the Parquet file lazily with arrow::open_dataset() rather than read_parquet(), then pipe a dplyr query: filter(mslevel == "1") to keep low-collision-energy MS1 scans, group_by(rt), and summarise(int = sum(intensity)), finishing with collect(). The rationale is that the on-disk Arrow engine pushes the filter/aggregation down to the columnar file, so only the aggregated result is materialized — the article reports ~1s runtime and only 64 Kb of resulting data for the example cal1-full-long.parquet (which occupies ~1.9 Gb if loaded fully in memory). For heavier aggregations, e.g. binned rt (floor(rt/0.1)*0.1) and mz (floor(mz/1)*1) heatmaps, route the query through DuckDB via to_duckdb() since the pure Arrow version takes ~30s to 1 min while DuckDB runs in a few seconds. The same pattern generalizes to EICs (filter on an m/z window such as mz > 216.0016 & mz < 216.2016) and IMS traces (group by bin and mz).

## Related tools

- **arcMS** (Converts Waters UNIFI (HD)MSE HRMS data into the Apache Parquet columnar files that this TIC query consumes) — https://github.com/leesulab/arcMS
- **Arrow** (Provides open_dataset() lazy on-disk querying so the TIC aggregation filters mslevel == 1 and sums intensity by rt without loading ~1.9 Gb into RAM) — https://github.com/leesulab/arcMS
- **Shiny** (Companion arcms-dataviz app retrieves TIC and spectra from Parquet files with Arrow for browser-based visualization) — https://github.com/leesulab/arcms-dataviz

## Examples

```
TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- The TIC query (filter mslevel == "1", group_by rt, summarise sum of intensity, collect) completes in around 1s on the example cal1-full-long.parquet.
- The collected TIC result occupies only ~64 Kb, demonstrating the aggregation ran on-disk rather than in memory.
- The full dataset loaded via read_parquet() occupies ~1.9 Gb in memory, confirming the same data was queried efficiently on disk.
- For binned rt/mz aggregations, switching to to_duckdb() should reduce runtime from ~30s-1 min (pure Arrow) to a few seconds.
- The Parquet source file for full profile-mode IMS HDMSE data is around 200-400 MB, sanity-checking that the input was produced by arcMS.

## Limitations

- Pure Arrow binned rt/mz aggregation queries are slow (~30s to 1 min); DuckDB streaming via to_duckdb() is required for interactive performance.
- read_parquet() loads the entire sample into memory (~1.9 Gb for one example HRMS sample), so lazy open_dataset() queries are preferred for chromatogram extraction.
- The workflow is specific to arcMS Parquet output (columns such as mslevel, rt, mz, intensity, scanid, bin); other HRMS formats like mzML require much slower read paths (15-20 min with DEIMoS, 1-2 min in MZmine).

## Evidence

- [methods] TIC via Arrow on-disk filter/group/sum: "TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()"
- [methods] Query time and output size: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] In-memory footprint of full data: "the loaded data for this example sample occupies ~1.9 Gb in memory"
- [methods] DuckDB acceleration for binned queries: [evidence span withheld — anchor status: not_found]
- [abstract] ArcMS stores data in Parquet for fast access: "providing functions to collect MSE data from the Waters UNIFI software and store it in the efficient Apache Parquet format, facilitating fast, easy access"
- [readme] Dataviz app retrieves chromatograms lazily: [evidence span withheld — anchor status: external_doc]
