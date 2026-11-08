---
name: long-format-ms-spectra-table-construction
description: Use when raw vendor HRMS acquisitions (Waters UNIFI (HD)MSE, or Thermo
  RAW converted to mzML) must be turned into an open, compact long-format table (rt,
  scanid, mslevel, mz, intensity, plus bin/dt for ion mobility) for downstream querying
  in R or Python.
when_to_use_negative:
- The data are already a feature/peak table or have already been converted to Parquet/HDF5
  with arcMS — use Arrow/DuckDB query-and-filter steps instead of re-conversion
- Only a visualization is needed from an existing arcMS Parquet file — the arcms-dataviz
  Shiny app can be applied directly without reconverting
- The source is neither UNIFI-accessible (HD)MSE data nor convertible to mzML (arcMS
  supports Waters UNIFI and mzML inputs, not arbitrary proprietary formats directly)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: arcMS
  role: 'R package that performs the conversion: collects UNIFI (HD)MSE data via the
    API or parses mzML, and writes the long-format Apache Parquet (or HDF5) table'
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Used to load and verify the resulting Parquet table (arrow::read_parquet)
    and to query/filter it on disk
- name: Proteowizard msconvert
  role: Converts vendor proprietary RAW files (e.g. Thermo) to open mzML format as
    the entry point for arcMS's mzML conversion path
- name: Waters UNIFI
  role: Source system whose Web API provides (HD)MSE sample data collected by arcMS
- name: R
  role: Language in which arcMS runs; converted tables can also be consumed from Python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:9b2453558f3939af13b7aad1b29d7e2b71044620f799306321640a851f856766
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - Proteowizard msconvert
  - R
  - Waters UNIFI
  - arcMS
  - data.table
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# long-format-ms-spectra-table-construction

## Summary

Convert high-resolution mass spectrometry data (Waters UNIFI (HD)MSE via the UNIFI API, or vendor files such as Thermo RAW first converted to mzML with Proteowizard msconvert) into a long-format columnar Apache Parquet table with the arcMS R package for compact storage and fast Arrow-based access.

## When to use

Use when raw vendor HRMS acquisitions (Waters UNIFI (HD)MSE, or Thermo RAW converted to mzML) must be turned into an open, compact long-format table (rt, scanid, mslevel, mz, intensity, plus bin/dt for ion mobility) for downstream querying in R or Python.

## When NOT to use

- The data are already a feature/peak table or have already been converted to Parquet/HDF5 with arcMS — use Arrow/DuckDB query-and-filter steps instead of re-conversion
- Only a visualization is needed from an existing arcMS Parquet file — the arcms-dataviz Shiny app can be applied directly without reconverting
- The source is neither UNIFI-accessible (HD)MSE data nor convertible to mzML (arcMS supports Waters UNIFI and mzML inputs, not arbitrary proprietary formats directly)

## Inputs

- Waters UNIFI (HD)MSE sample/analysis accessible via the UNIFI Web API (API host URL and identity token URL)
- mzML file obtained from vendor RAW data (e.g. Thermo RAW converted with Proteowizard msconvert)
- Optional ion mobility metadata (drift time bin index, dt, CCS)

## Outputs

- converted_file.parquet: long-format Apache Parquet table with columns rt, scanid, mslevel, mz, intensity (plus bin and dt for IMS data)
- sample_name-metadata. metadata file (also embedded in the Parquet file)
- Optionally file.h5 in HDF5 format with ms1/ms2/metadata datasets

## How to apply

For Waters UNIFI data, first configure the UNIFI Web API and create connection parameters with create_connection_params(apihosturl, identityurl); then use convert_one_sample_data(sample_id) or convert_all_samples_data(analysis_id) — optionally with the future multisession plan for parallel collection — which writes sample_name.parquet plus a sample_name-metadata.. For other vendors (e.g. Thermo RAW), convert the RAW file to mzML with Proteowizard msconvert and parse it with arcMS's mzML functions (mzml_to_dt / mzml_to_sample_dataset with chunked reading, then convert_mzml_to_parquet), mirroring the same column contract as Waters files. Each row of the output is one spectral point with columns rt, scanid, mslevel, mz and intensity, extended with bin and dt when ion mobility (HDMS(E)) data are present. Choose Parquet for minimal filesize and fast access (HDF5 via format="hdf5" is the alternative); optionally retrieve CCS values in addition to bin index and drift time (convert_bin_to_ccs). Validate by loading with arrow::read_parquet and comparing the schema against the reference file cal1-full-long.parquet.

## Related tools

- **arcMS** (R package that performs the conversion: collects UNIFI (HD)MSE data via the API or parses mzML, and writes the long-format Apache Parquet (or HDF5) table) — https://github.com/leesulab/arcMS
- **Arrow** (Used to load and verify the resulting Parquet table (arrow::read_parquet) and to query/filter it on disk)
- **Proteowizard msconvert** (Converts vendor proprietary RAW files (e.g. Thermo) to open mzML format as the entry point for arcMS's mzML conversion path)
- **Waters UNIFI** (Source system whose Web API provides (HD)MSE sample data collected by arcMS)
- **R** (Language in which arcMS runs; converted tables can also be consumed from Python)

## Examples

```
con = create_connection_params(apihosturl = "http://localhost:50034/unifi/v1", identityurl = "http://localhost:50333/identity/connect/token"); convert_one_sample_data(sample_id = "0134efbf-c75a-411b-842a-4f35e2b76347")
```

## Evaluation signals

- arrow::read_parquet('converted_file.parquet') loads successfully without errors
- Schema contains the required spectral columns rt, scanid, mslevel, mz and intensity (with bin and dt present for IMS data)
- Output schema matches the reference Waters-UNIFI Parquet contract of cal1-full-long.parquet
- Companion JSON metadata file is produced and metadata is also stored inside the Parquet file
- Expected file size: full profile-mode IMS HDMS(E) data in Parquet is around 200-400 MB (compactness sanity check)

## Limitations

- Full profile-mode IMS HDMS(E) data loads to ~1.9 Gb in memory; avoid full in-memory reads — prefer on-disk Arrow queries (a TIC aggregation retrieves only ~64 Kb)
- mz binning/aggregation queries in pure Arrow can take ~30s to 1 min; DuckDB (to_duckdb) runs the same in a few seconds
- arcMS natively collects Waters UNIFI data; other vendors must be routed through mzML conversion (e.g. Proteowizard msconvert) first
- HDF5 output requires the rhdf5 package to be installed

## Evidence

- [readme] mzML support: "Conversion of mzML files is also supported (see `convert_mzml_to_parquet()`)."
- [intro] ion mobility compatibility: "It is compatible with data containing ion mobility
(HDMS<sup>E</sup>) or not (MS<sup>E</sup>). Conversion of mzML files is
also supported"
- [methods] conversion function: "convert_one_sample_data(sample_id = "0134efbf-c75a-411b-842a-4f35e2b76347")"
- [methods] validation read: "data = read_parquet("converted_file.parquet")"
- [methods] reference dataset: "cal1-full-long.parquet", "cal1.parquet")"
