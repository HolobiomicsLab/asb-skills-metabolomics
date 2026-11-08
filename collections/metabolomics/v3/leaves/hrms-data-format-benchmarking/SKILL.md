---
name: hrms-data-format-benchmarking
description: Use when benchmarking mass spectrometry data storage formats (Parquet,
  mzML, mz5, mzMLb, HDF5) for conversion speed, file size, or read/query latency on
  Waters UNIFI (HD)MSE or mzML-derived HRMS data.
when_to_use_negative:
- The data are already stored in Parquet or HDF5 and no cross-format comparison is
  requested — go straight to querying instead of re-benchmarking.
- The source data cannot be exported from Waters UNIFI (or as mzML), since arcMS only
  collects (HD)MSE data via the UNIFI API or converts mzML files via convert_mzml_to_parquet().
- The goal is a one-off visualization only — the arcms-dataviz Shiny app reads Parquet
  directly without any benchmarking overhead.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: arcMS
  role: R package that collects (HD)MSE data from Waters UNIFI and converts it to
    Apache Parquet (or HDF5) for the benchmarked fast-access workflow
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: R library used to read Parquet files in memory (read_parquet) and query them
    on-disk (open_dataset + dplyr filtering/aggregation) when timing read speed and
    data reduction
  repo: https://arrow.apache.org/
- name: Python
  role: Alternative language for opening the converted Parquet data (per the open-files
    vignette), demonstrating cross-language compatibility during benchmarking
- name: Shiny
  role: arcms-dataviz companion R/Shiny app that retrieves chromatograms and MS spectra
    from Parquet on-disk via Arrow, illustrating the read-speed advantage being benchmarked
  repo: https://github.com/leesulab/arcms-dataviz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:db3051e0aa6f439f5c9609a314ae94f6dc67b821bce48de1c090c53854bd9292
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hrms-data-format-benchmarking

## Summary

Reproduce and extend the cross-format benchmarking of HRMS data storage/conversion workflows described in the arcMS paper: compare Apache Parquet (via arcMS) against mz5, mzML, gzipped mzML, and mzMLb produced with MSConvertGUI in terms of conversion time, on-disk file size, and read/query speed.

## When to use

Use when benchmarking mass spectrometry data storage formats (Parquet, mzML, mz5, mzMLb, HDF5) for conversion speed, file size, or read/query latency on Waters UNIFI (HD)MSE or mzML-derived HRMS data.

## When NOT to use

- The data are already stored in Parquet or HDF5 and no cross-format comparison is requested — go straight to querying instead of re-benchmarking.
- The source data cannot be exported from Waters UNIFI (or as mzML), since arcMS only collects (HD)MSE data via the UNIFI API or converts mzML files via convert_mzml_to_parquet().
- The goal is a one-off visualization only — the arcms-dataviz Shiny app reads Parquet directly without any benchmarking overhead.

## Inputs

- cal1-full-long.parquet example HRMS file from the leesulab/arcms-dataviz repository (GitHub media URL given in the methods)
- Waters UNIFI (HD)MSE sample data or mzML files convertible via arcMS/MSConvertGUI
- Article-reported benchmark values from the Results section and Fig. 1 (conversion times, file sizes, read times)

## Outputs

- benchmark_table.csv with columns: format, conversion time, file size, read time
- Measured Arrow query timings (TIC ~1s / ~64 Kb; binned rt/mz aggregation timings for Arrow vs DuckDB)
- In-memory footprint measurement (~1.9 Gb) of the loaded Parquet example file

## How to apply

Download the example HRMS file cal1-full-long.parquet from leesulab/arcms-dataviz and load it in R with arrow::read_parquet() to verify the in-memory footprint (~1.9 Gb reported); then reopen it on-disk with arrow::open_dataset() and time a dplyr query filtering mslevel == "1", grouping by rt and summing intensity to obtain the TIC — it should collect in around 1s and yield only ~64 Kb. Time the binned rt/mz aggregation query (rt_binned = floor(rt/0.1)*0.1, mz_binned = floor(mz/1)*1) via pure Arrow (~30s to 1 min) versus via to_duckdb() streaming (a few seconds) to characterize query backends. Then gather article-reported benchmark values: arcMS converted 10 HRMS samples in 1–3 min vs MSConvertGUI requiring 11–18 min for mz5, mzML, or gzipped mzML (~7 min for mzMLb); file sizes of Parquet 80–600 MB (200–400 MB for full profile-mode IMS HDMSE) vs uncompressed mzML 5–6 GB, gzipped mzML/mz5 300 MB–1 GB, mzMLb ~200–600 MB; and read times of Parquet 1–3 s vs mzML/DEIMoS 15–20 min and MZmine 1–2 min. Assemble measured and reported values into a structured table with columns for format, conversion time, file size, and read time (benchmark_table.csv).

## Related tools

- **arcMS** (R package that collects (HD)MSE data from Waters UNIFI and converts it to Apache Parquet (or HDF5) for the benchmarked fast-access workflow) — https://github.com/leesulab/arcMS
- **Arrow** (R library used to read Parquet files in memory (read_parquet) and query them on-disk (open_dataset + dplyr filtering/aggregation) when timing read speed and data reduction) — https://arrow.apache.org/
- **Python** (Alternative language for opening the converted Parquet data (per the open-files vignette), demonstrating cross-language compatibility during benchmarking)
- **Shiny** (arcms-dataviz companion R/Shiny app that retrieves chromatograms and MS spectra from Parquet on-disk via Arrow, illustrating the read-speed advantage being benchmarked) — https://github.com/leesulab/arcms-dataviz

## Examples

```
data_arrow = arrow::open_dataset("cal1-full-long.parquet"); TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- The Arrow TIC query (filter mslevel == "1", group_by rt, sum intensity, collect) completes in approximately 1s and returns only ~64 Kb of data.
- The fully loaded cal1-full-long.parquet occupies around 1.9 Gb in memory when read with arrow::read_parquet().
- The DuckDB-backed binned rt/mz aggregation (via to_duckdb()) runs in a few seconds, whereas the pure Arrow query takes ~30s to 1 min.
- benchmark_table.csv reproduces the reported ranges: arcMS 1–3 min vs MSConvertGUI 11–18 min conversion; Parquet 80–600 MB (200–400 MB for full profile-mode IMS HDMSE) vs mzML 5–6 GB; Parquet reads 1–3 s vs mzML 15–20 min.
- Parquet files include both low- and high-energy (HDMSE) spectra plus a companion sample-metadata., matching the arcMS output contract.

## Limitations

- arcMS relies on proper configuration of the UNIFI Web API (connection parameters with API host and identity token URLs); data on another machine requires replacing localhost with the API IP address.
- The pure Arrow binned rt/mz aggregation query is slow (~30s to 1 min); DuckDB streaming is required to bring this to a few seconds.
- HDF5 output offers fast access but a larger filesize than Parquet, so HDF5 should not be expected to match Parquet's compact-storage benchmark numbers.
- The article reports no changelog for the arcMS repository, so tool version drift may affect reproducibility of timings.

## Evidence

- [abstract] arcMS collects MSE data from Waters UNIFI and stores it in Apache Parquet: "providing functions to collect MSE data from the Waters UNIFI software and store it in the efficient Apache Parquet format"
- [methods] TIC on-disk Arrow query timing and size: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] In-memory footprint of the example sample: "the loaded data for this example sample occupies \~1.9 Gb in memory"
- [methods] Arrow vs DuckDB binned aggregation timing: [evidence span withheld — anchor status: not_found]
- [readme] On-disk Arrow access rationale from the dataviz app README: [evidence span withheld — anchor status: external_doc]
