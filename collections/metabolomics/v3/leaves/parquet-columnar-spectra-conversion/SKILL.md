---
name: parquet-columnar-spectra-conversion
description: Use when proprietary vendor HRMS data (Waters UNIFI (HD)MSE, or Thermo
  RAW converted to mzML) must be stored compactly, queried fast, or shared across
  R and Python.
when_to_use_negative:
- The data is already a processed feature table or peak list rather than raw/profile
  HRMS spectra
- Only a single chromatogram or spectrum plot is needed — the arcms-dataviz Shiny
  app can visualize existing Parquet files without re-conversion
- The source is not UNIFI-accessible data nor a format convertible to mzML (arcMS's
  documented ingestion paths)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3077
tools:
- name: arcMS
  role: R package that converts (HD)MSE and mzML data to Parquet/HDF5 columnar format;
    provides convert_mzml_to_parquet, mzml_to_dt, mzml_to_sample_dataset, convert_one_sample_data,
    convert_all_samples_data
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Reads and queries the resulting Parquet files on disk without full in-RAM
    loading (arrow::read_parquet, dplyr filters)
- name: Proteowizard msconvert
  role: Converts vendor RAW files (e.g. Thermo) to open mzML format before arcMS mzML
    ingestion
- name: Waters UNIFI
  role: Source software whose (HD)MSE data is retrieved via the UNIFI Web API for
    conversion
- name: rhdf5
  role: Optional dependency used when the HDF5 output format is requested instead
    of Parquet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:b3f29d9c007ff2b5fd998b11c323624c0d6a9313aebe02c5fda730cef7e7e0fe
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
  - rhdf5
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# parquet-columnar-spectra-conversion

## Summary

Convert high-resolution mass spectrometry data — Waters UNIFI (HD)MSE or vendor RAW files intermediated as mzML (e.g. Thermo) — into a long-format Apache Parquet columnar table with columns rt, scanid, mslevel, mz and intensity (plus bin/dt for ion mobility), for compact storage and fast Arrow-based access.

## When to use

Use when proprietary vendor HRMS data (Waters UNIFI (HD)MSE, or Thermo RAW converted to mzML) must be stored compactly, queried fast, or shared across R and Python.

## When NOT to use

- The data is already a processed feature table or peak list rather than raw/profile HRMS spectra
- Only a single chromatogram or spectrum plot is needed — the arcms-dataviz Shiny app can visualize existing Parquet files without re-conversion
- The source is not UNIFI-accessible data nor a format convertible to mzML (arcMS's documented ingestion paths)

## Inputs

- Waters UNIFI (HD)MSE sample data accessed through the UNIFI Web API (sample_id or analysis_id)
- Vendor RAW files from other instruments (e.g. Thermo) converted to open mzML format with Proteowizard msconvert
- UNIFI API connection parameters (API host URL and identity token URL)

## Outputs

- converted_file.parquet — long-format Apache Parquet table with columns rt, scanid, mslevel, mz, intensity (plus bin and dt for IMS data)
- sample_name-metadata. sidecar metadata (also embedded in the Parquet file)
- Optionally file.h5 in HDF5 format with ms1/ms2/metadata datasets when format = 'hdf5' is requested

## How to apply

For Waters UNIFI data, configure the UNIFI Web API (create_connection_params with apihosturl and identityurl), search folders/analyses with folders_search and analysis_search, then convert one or all samples with convert_one_sample_data or convert_all_samples_data. For other vendors (e.g. Thermo RAW), first convert the RAW file to open mzML with Proteowizard msconvert using the same configuration arcMS uses for Waters data, then parse the mzML with arcMS (mzml_to_dt / mzml_to_sample_dataset, chunked reading) and write converted_file.parquet via convert_mzml_to_parquet. The output long-format table carries columns rt, scanid, mslevel, mz and intensity, with bin and dt added when ion mobility (HDMSE) data are present. The Parquet format is chosen for minimal filesize and fast access (HDF5 is the alternative when larger files are acceptable). Validate by loading with arrow::read_parquet and comparing the schema to the reference Waters-UNIFI file cal1-full-long.parquet.

## Related tools

- **arcMS** (R package that converts (HD)MSE and mzML data to Parquet/HDF5 columnar format; provides convert_mzml_to_parquet, mzml_to_dt, mzml_to_sample_dataset, convert_one_sample_data, convert_all_samples_data) — https://github.com/leesulab/arcMS
- **Arrow** (Reads and queries the resulting Parquet files on disk without full in-RAM loading (arrow::read_parquet, dplyr filters))
- **Proteowizard msconvert** (Converts vendor RAW files (e.g. Thermo) to open mzML format before arcMS mzML ingestion)
- **Waters UNIFI** (Source software whose (HD)MSE data is retrieved via the UNIFI Web API for conversion)
- **rhdf5** (Optional dependency used when the HDF5 output format is requested instead of Parquet)

## Examples

```
con = create_connection_params(apihosturl = "http://localhost:50034/unifi/v1", identityurl = "http://localhost:50333/identity/connect/token"); convert_one_sample_data(sample_id = "0134efbf-c75a-411b-842a-4f35e2b76347")
```

## Evaluation signals

- arrow::read_parquet('converted_file.parquet') loads successfully without error
- Schema contains the required spectral columns rt, scanid, mslevel, mz and intensity, with bin and dt present for IMS/HDMSE data
- Column contract matches the reference Waters-UNIFI Parquet file cal1-full-long.parquet
- A metadata JSON sidecar (sample_name-metadata.) is produced alongside the Parquet file, and metadata is also stored inside the Parquet file
- Parquet file size is compact: full profile-mode IMS HDMSE files land around 200-400 MB

## Limitations

- UNIFI API must be properly configured and the client application registered before arcMS can connect; localhost must be replaced by the API machine's IP when running remotely
- HDF5 output has fast access but larger filesize than Parquet
- Loaded example HRMS data occupies ~1.9 Gb in memory, so full in-RAM loading is costly; on-disk Arrow queries are preferred
- Native ingestion beyond Waters UNIFI requires an intermediate mzML conversion step (e.g. for Thermo RAW files)

## Evidence

- [abstract] arcMS converts Waters (HD)MSE and mzML to Parquet: "providing functions to collect MSE data from the Waters UNIFI software and store it in the efficient Apache Parquet format"
- [readme] README: mzML conversion supported: "Conversion of mzML files is
also supported (see `convert_mzml_to_parquet()`)."
- [readme] README: connection parameters: "con = create_connection_params(apihosturl = "http://localhost:50034/unifi/v1", identityurl = "http://localhost:50333/identity/connect/token")"
- [methods] methods: load Parquet with arrow: "data = read_parquet("converted_file.parquet")"
- [intro] intro: output formats: "the [Apache Parquet](https://parquet.apache.org/) format for minimal
  filesize and fast access."
- [methods] methods: file size range: "A full data file in profile mode containing IMS, and HDMS<sup>E</sup> data (MS and MS/MS), is around 200-400 Mo."
