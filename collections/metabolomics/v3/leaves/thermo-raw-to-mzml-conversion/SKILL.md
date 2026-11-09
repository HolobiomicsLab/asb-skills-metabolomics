---
name: thermo-raw-to-mzml-conversion
description: Use when the input is a Thermo (or other vendor) proprietary RAW file
  from a mass spectrometry repository and the goal is an open, Parquet-compatible
  spectral table matching the arcMS Waters-UNIFI schema (rt, scanid, mslevel, mz,
  intensity).
when_to_use_negative:
- Input is already an arcMS Parquet or mzML file — conversion of RAW to mzML is unnecessary
- Data originates from Waters UNIFI — use arcMS's native UNIFI API collection (create_connection_params
  / convert_one_sample_data) instead
- The goal is direct in-memory analysis without a persisted columnar file — HDF5 output
  via arcMS may be preferable when large filesize is acceptable
edam_operation: http://edamontology.org/operation_0324
edam_topics: []
tools:
- name: arcMS
  role: R package providing convert_mzml_to_parquet() and mzml_to_dt / mzml_to_sample_dataset
    (chunked reading) to parse mzML and write the columnar Parquet table
  repo: https://github.com/leesulab/arcMS
- name: Proteowizard msconvert
  role: Converts the vendor Thermo RAW file into the open mzML format that arcMS can
    ingest
- name: Arrow
  role: Loads and validates the resulting Parquet file (arrow::read_parquet) and enables
    fast on-disk queries/filtration
  repo: https://arrow.apache.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:7171add2c06d24ecc60ebececc3e969048bd463eba80d2cd7ed55e1c904a20bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - Proteowizard msconvert
  - R
  - arcMS
  edam_operation: http://edamontology.org/operation_0324
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# thermo-raw-to-mzml-conversion

## Summary

Extends the arcMS columnar-conversion approach beyond Waters UNIFI (HD)MSE data to Thermo proprietary RAW files by first converting the vendor RAW file to open mzML with Proteowizard msconvert, then using arcMS mzML functions to write a long-format Apache Parquet table.

## When to use

Use when the input is a Thermo (or other vendor) proprietary RAW file from a mass spectrometry repository and the goal is an open, Parquet-compatible spectral table matching the arcMS Waters-UNIFI schema (rt, scanid, mslevel, mz, intensity).

## When NOT to use

- Input is already an arcMS Parquet or mzML file — conversion of RAW to mzML is unnecessary
- Data originates from Waters UNIFI — use arcMS's native UNIFI API collection (create_connection_params / convert_one_sample_data) instead
- The goal is direct in-memory analysis without a persisted columnar file — HDF5 output via arcMS may be preferable when large filesize is acceptable

## Inputs

- Thermo RAW proprietary mass spectrometry file (e.g. downloaded from a public MS repository)
- mzML file produced by Proteowizard msconvert

## Outputs

- converted_file.parquet — Apache Parquet long-format table with columns rt, scanid, mslevel, mz, intensity (plus bin and dt for IMS data)
- Optional sample metadata JSON alongside the Parquet file

## How to apply

Download a publicly deposited Thermo RAW file and convert it to mzML with Proteowizard msconvert, mirroring the configuration arcMS uses for Waters UNIFI data, since arcMS supports mzML as an input format. Parse the mzML with arcMS mzML functions (mzml_to_dt / mzml_to_sample_dataset with chunked reading) to build a long-format table with columns rt, scanid, mslevel, mz and intensity, plus bin and dt when ion mobility (HDMSE) data are present. Write the table to converted_file.parquet using arcMS (e.g. convert_mzml_to_parquet()); Parquet is preferred over HDF5 for minimal filesize and fast access. Finally, load the output with arrow::read_parquet and compare its schema against the reference Waters-UNIFI file cal1-full-long.parquet to confirm the column contract matches.

## Related tools

- **arcMS** (R package providing convert_mzml_to_parquet() and mzml_to_dt / mzml_to_sample_dataset (chunked reading) to parse mzML and write the columnar Parquet table) — https://github.com/leesulab/arcMS
- **Proteowizard msconvert** (Converts the vendor Thermo RAW file into the open mzML format that arcMS can ingest)
- **Arrow** (Loads and validates the resulting Parquet file (arrow::read_parquet) and enables fast on-disk queries/filtration) — https://arrow.apache.org/

## Examples

```
msconvert thermo_sample.RAW --mzML; then in R: arcMS::convert_mzml_to_parquet("thermo_sample.mzML"); arrow::read_parquet("converted_file.parquet")
```

## Evaluation signals

- arrow::read_parquet('converted_file.parquet') loads successfully without error
- Schema contains the required spectral columns rt, scanid, mslevel, mz and intensity (with bin/dt for IMS data)
- Output schema matches the reference Waters-UNIFI Parquet contract (cal1-full-long.parquet)
- Full profile-mode IMS HDMSE Parquet files should be on the order of ~200-400 MB
- TIC aggregation via an on-disk Arrow query returns only ~64 Kb of data, confirming the columnar layout supports fast partial reads

## Limitations

- Thermo RAW conversion requires an intermediate msconvert step; direct UNIFI-API-style collection is only available for Waters data
- HDF5 output is possible but produces larger files than Parquet
- No changelog is available for the arcMS repository, so version-specific behaviour of the mzML conversion functions is harder to track

## Evidence

- [readme] mzML conversion support: "Conversion of mzML files is also supported (see `convert_mzml_to_parquet()`)."
- [intro] ion mobility compatibility: "It is compatible with data containing ion mobility (HDMS<sup>E</sup>) or not (MS<sup>E</sup>). Conversion of mzML files is also supported"
- [intro] core conversion claim: "`arcMS` can convert (HD)MS<sup>E</sup> data acquired with Waters UNIFI
to tabular format for use in R or Python, with a small filesize when
saved on disk"
- [methods] Parquet storage of HRMS data: "HRMS data converted with `arcMS` is stored as columnar files in the Apache Parquet format"
- [readme] Parquet output format rationale: "the [Apache Parquet](https://parquet.apache.org/) format for minimal
  filesize and fast access."
