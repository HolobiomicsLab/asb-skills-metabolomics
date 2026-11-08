---
name: chromatogram-extraction-from-parquet-ms-data
description: Use when a chromatogram (TIC, BPI, or EIC for a specific m/z window)
  must be computed from an on-disk Parquet HRMS file too large to load fully into
  memory (e.g. ~1.9 Gb in-RAM, 200-400 MB Parquet profile data).
when_to_use_negative:
- The data are already fully loaded in R memory as a regular data frame and small
  enough for dplyr/Arrow in-memory aggregation
- The input is a proprietary UNIFI or mzML raw file that must first be converted to
  Parquet/HDF5 with arcMS
- The output format is HDF5 (ms1/ms2 datasets) rather than Parquet — this query path
  targets the Arrow/Parquet columnar representation
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: arcMS
  role: Produces the Parquet-format HRMS files that this chromatogram-extraction query
    consumes
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Opens the Parquet dataset on-disk with open_dataset() for RAM-bounded filtering
    and aggregation
- name: DuckDB
  role: Receives the Arrow dataset via zero-copy to_duckdb() and accelerates complex
    filter/aggregation SQL to a few seconds
- name: arcms-dataviz
  role: R/Shiny companion app that visualizes TIC, BPI or EIC chromatograms retrieved
    from the Parquet file with Arrow
  repo: https://github.com/leesulab/arcms-dataviz
- name: dplyr
  role: Expresses the filter/group_by/summarise/arrange steps of the chromatogram
    query
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:e1a20ed9f866b67bc0c76c1c8d841e3d3d42b8d0f4228aeffc49c08cc3c27039
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - DuckDB
  - R
  - arcMS
  - arcms-dataviz
  - data.table
  - dplyr
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# chromatogram-extraction-from-parquet-ms-data

## Summary

Extract EIC/TIC chromatograms from Parquet-stored profile-mode HRMS data by streaming an Arrow dataset into DuckDB and running dplyr/SQL filter-group-summarise queries without loading the file into RAM.

## When to use

Use when a chromatogram (TIC, BPI, or EIC for a specific m/z window) must be computed from an on-disk Parquet HRMS file too large to load fully into memory (e.g. ~1.9 Gb in-RAM, 200-400 MB Parquet profile data).

## When NOT to use

- The data are already fully loaded in R memory as a regular data frame and small enough for dplyr/Arrow in-memory aggregation
- The input is a proprietary UNIFI or mzML raw file that must first be converted to Parquet/HDF5 with arcMS
- The output format is HDF5 (ms1/ms2 datasets) rather than Parquet — this query path targets the Arrow/Parquet columnar representation

## Inputs

- On-disk Apache Parquet file of arcMS-converted (HD)MSE HRMS data (e.g. cal1-full-long.parquet)
- Target analyte m/z and extraction window width (e.g. +/-0.1 Da around atrazine m/z 216.1016)

## Outputs

- Small aggregated data frame/table with rt and summed intensity columns (e.g. eic_atrazine.csv)
- Optionally MS spectra or binned rt/mz aggregates produced by analogous filter-group-summarise queries

## How to apply

Open the Parquet file on-disk with arrow::open_dataset() so the ~200-400 MB profile-mode HDMSE file is never fully read into RAM. Stream the Arrow dataset into DuckDB via to_duckdb() for zero-copy SQL querying — this turns a ~30s-to-1-min pure-Arrow binned aggregation into a query that runs in a few seconds. For an EIC, filter to mslevel == '1' (low collision energy) and window the m/z to +/-0.1 around the target (e.g. 216.0016 < mz < 216.2016 for atrazine, m/z 216.1016), then group_by(rt, scanid) and summarise(intensity = sum(intensity)) in dplyr syntax; only collect() the resulting small table. Arrange by increasing retention time and save as CSV. For a TIC, apply the same pattern without the m/z window — an on-disk TIC query takes around 1s and yields only ~64 Kb of data.

## Related tools

- **arcMS** (Produces the Parquet-format HRMS files that this chromatogram-extraction query consumes) — https://github.com/leesulab/arcMS
- **Arrow** (Opens the Parquet dataset on-disk with open_dataset() for RAM-bounded filtering and aggregation)
- **DuckDB** (Receives the Arrow dataset via zero-copy to_duckdb() and accelerates complex filter/aggregation SQL to a few seconds)
- **arcms-dataviz** (R/Shiny companion app that visualizes TIC, BPI or EIC chromatograms retrieved from the Parquet file with Arrow) — https://github.com/leesulab/arcms-dataviz
- **dplyr** (Expresses the filter/group_by/summarise/arrange steps of the chromatogram query)

## Examples

```
eic = data_arrow |> to_duckdb() |> filter(mslevel == "1") |> filter(mz > 216.0016 & mz < 216.2016) |> group_by(rt, scanid) |> summarise(intensity = sum(intensity)) |> collect()
```

## Evaluation signals

- Output CSV exists and contains exactly the rt and intensity columns
- All summed intensity values are positive
- Result table is small (on-disk aggregation; TIC equivalent produced only ~64 Kb)
- Rows are ordered by increasing retention time
- Query is verified to not load the full ~1.9 Gb dataset into RAM (dataset was opened with arrow::open_dataset, not read_parquet)

## Limitations

- Pure-Arrow binned rt/mz aggregation queries can take ~30s to 1 min; DuckDB is needed to reduce this to a few seconds
- This query path applies to Parquet outputs; HDF5 outputs (ms1/ms2 datasets) use rhdf5::h5read instead
- Large profile-mode files occupy ~1.9 Gb when loaded with read_parquet, so full in-memory loading defeats the purpose of the on-disk workflow

## Evidence

- [methods] On-disk DuckDB-accelerated EIC query: [evidence span withheld — anchor status: not_found]
- [methods] EIC filter and aggregation expression: "filter(mz > 216.0016 & mz < 216.2016) |> 
  group_by(rt, scanid) |>
  summarise(intensity = sum(intensity))"
- [methods] DuckDB speedup over pure Arrow: "The resulting object is now just several Mbs, but the query takes \~30s to 1 min"
- [methods] TIC on-disk aggregation cost: "This took around 1s and the resulting data is only 64 Kb!"
- [readme] README: chromatograms from Parquet without full RAM load: [evidence span withheld — anchor status: external_doc]
- [methods] Loaded example HRMS data occupies ~1.9 Gb in memory: "the loaded data for this example sample occupies \~1.9 Gb in memory"
