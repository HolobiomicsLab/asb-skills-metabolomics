---
name: profile-mode-hrms-data-handling
description: Use when raw profile-mode HRMS files (Waters UNIFI MSE/HDMSE exports
  or mzML) must be converted to a compact columnar format while retaining ion-mobility
  dimensions, and later queried, filtered or aggregated with Apache Arrow.
when_to_use_negative:
- The data are already stored as an arcMS-created Parquet file and the goal is visualization
  or filtering — use the Arrow/duckdb query pipeline or the arcms-dataviz Shiny app
  instead of re-converting.
- The source data are not Waters UNIFI (HD)MSE exports or mzML files — arcMS collects
  MSE from UNIFI and supports mzML conversion, not other vendor formats.
- Only the HDF5 layout (ms1/ms2/metadata datasets in one .h5 file) is required and
  Parquet is not acceptable — although arcMS supports format="hdf5", this skill targets
  the Parquet/Arrow columnar workflow.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: arcMS
  role: R package that chunks mzML/UNIFI data into the long-format table and writes
    Parquet (or HDF5) with metadata via convert_mzml_to_parquet() and convert_one_sample_data()
  repo: https://github.com/leesulab/arcMS
- name: Apache Arrow
  role: On-disk lazy querying, filtering and aggregation of the Parquet dataset (filter(mslevel
    == "1"), group_by(rt), summarise(sum(intensity)))
  repo: https://arrow.apache.org/
- name: DEIMoS
  role: Reference loader whose deimos.load() DataFrame is compared element-wise against
    the Parquet DataFrame to validate lossless conversion fidelity
- name: pandas
  role: read_parquet() loads the converted file in Python for the DataFrame-identity
    fidelity check
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:7e462812af4adaf165b0d04330647a40c6f0627d1296c9ce710084a89f8fc7db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Apache Arrow
  - DEIMoS
  - R
  - arcMS
  - data.table
  - mzR
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# profile-mode-hrms-data-handling

## Summary

Convert and handle multi-dimensional profile-mode HRMS data (MSE/HDMSE from Waters UNIFI, or mzML) into columnar Apache Parquet files with arcMS, preserving long-format spectra (rt, scanid, mslevel, mz, intensity, bin, dt) plus metadata, for compact storage and fast Arrow-based access in R or Python.

## When to use

Use when raw profile-mode HRMS files (Waters UNIFI MSE/HDMSE exports or mzML) must be converted to a compact columnar format while retaining ion-mobility dimensions, and later queried, filtered or aggregated with Apache Arrow.

## When NOT to use

- The data are already stored as an arcMS-created Parquet file and the goal is visualization or filtering — use the Arrow/duckdb query pipeline or the arcms-dataviz Shiny app instead of re-converting.
- The source data are not Waters UNIFI (HD)MSE exports or mzML files — arcMS collects MSE from UNIFI and supports mzML conversion, not other vendor formats.
- Only the HDF5 layout (ms1/ms2/metadata datasets in one .h5 file) is required and Parquet is not acceptable — although arcMS supports format="hdf5", this skill targets the Parquet/Arrow columnar workflow.

## Inputs

- Profile-mode mzML file (MSE or HDMSE, with or without ion mobility)
- Waters UNIFI sample/analysis data accessed via the UNIFI Web API (API host and identity token connection parameters)
- Apache Parquet file previously created with arcMS (for query/filter/aggregation workflows)

## Outputs

- converted_file.parquet — long-format columnar table with columns rt, scanid, mslevel, mz, intensity, and bin/dt when IMS data are present
- sample_name-metadata. sidecar plus metadata embedded in the Parquet key-value metadata
- Fidelity check report (e.g. fidelity_check.csv) comparing DEIMoS-loaded vs Parquet-loaded DataFrames
- Arrow query results such as TIC, EIC, or IMS trace aggregations (~64 Kb for a TIC)

## How to apply

Read the input profile-mode mzML in chunks via arcMS's convert_mzml_to_parquet() (chunksize bounds memory), exploding each scan into a long-format table with one row per m/z datapoint carrying scanid, rt, mslevel (MS1 vs high collision energy), mz and intensity; where ion mobility is present, keep the drift-time bin index (bin) and drift time (dt) so HDMSE dimensions survive. Write the table to converted_file.parquet with the Apache Arrow writer, saving sample and spectrum metadata both as a JSON sidecar (sample_name-metadata.) and inside the Parquet key-value metadata. Validate losslessness by loading the same mzML with deimos.load() and the Parquet with pandas read_parquet(), then asserting element-wise equality (DataFrame.equals / assert_frame_equal on rows, columns, dtypes, values). Rationale: Parquet minimizes filesize (~200-400 MB for a full profile IMS HDMSE file vs ~1.9 Gb in memory) and enables on-disk Arrow queries, e.g. a TIC aggregation returning only ~64 Kb; for heavy rt/mz-binned aggregations route the Arrow dataset through DuckDB (to_duckdb()) since pure Arrow runs ~30s-1min while DuckDB finishes in seconds.

## Related tools

- **arcMS** (R package that chunks mzML/UNIFI data into the long-format table and writes Parquet (or HDF5) with metadata via convert_mzml_to_parquet() and convert_one_sample_data()) — https://github.com/leesulab/arcMS
- **Apache Arrow** (On-disk lazy querying, filtering and aggregation of the Parquet dataset (filter(mslevel == "1"), group_by(rt), summarise(sum(intensity)))) — https://arrow.apache.org/
- **DEIMoS** (Reference loader whose deimos.load() DataFrame is compared element-wise against the Parquet DataFrame to validate lossless conversion fidelity)
- **pandas** (read_parquet() loads the converted file in Python for the DataFrame-identity fidelity check)

## Examples

```
library(arcMS); convert_one_sample_data(sample_id = "0134efbf-c75a-411b-842a-4f35e2b76347")
```

## Evaluation signals

- arrow::read_parquet or arrow::open_dataset opens converted_file.parquet as a valid dataset exposing the expected columns rt, scanid, mslevel, mz, intensity, bin, dt
- A TIC query — filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect() — succeeds and returns only ~64 Kb
- The DEIMoS-loaded DataFrame and the pandas read_parquet() DataFrame match exactly on row count, column names, dtypes and values (DataFrame.equals / assert_frame_equal)
- Metadata are present both in the JSON sidecar (importable with lite::fromJSON) and inside the Parquet key-value metadata
- File size falls in the expected ~200-400 MB range for a full profile-mode IMS HDMSE file (vs ~1.9 Gb once loaded in memory)

## Limitations

- Full profile-mode IMS HDMSE files are still ~200-400 MB on disk, and the loaded data for one example sample occupies ~1.9 Gb in memory
- Pure Arrow rt/mz binned aggregation queries take ~30s to 1 min; DuckDB (via to_duckdb()) is needed to run them in a few seconds
- UNIFI API access requires prior configuration and registration of a client application (see vignette("api-configuration")); localhost must be replaced by the API IP address when run from another machine
- No changelog is available for the arcMS repository, so version-specific behavior changes cannot be tracked

## Evidence

- [intro] It is compatible with data containing ion mobility (HDMSE) or not (MSE). Conversion of mzML files is also supported: "It is compatible with data containing ion mobility
(HDMS<sup>E</sup>) or not (MS<sup>E</sup>). Conversion of mzML files is
also supported"
- [abstract] store it in the efficient Apache Parquet format: "providing functions to collect MSE data from the Waters UNIFI software and store it in the efficient Apache Parquet format"
- [methods] HRMS data converted with `arcMS` is stored as columnar files in the Apache Parquet format: "HRMS data converted with `arcMS` is stored as columnar files in the Apache Parquet format"
- [methods] the loaded data for this example sample occupies ~1.9 Gb in memory: "the loaded data for this example sample occupies \~1.9 Gb in memory"
- [readme] Conversion of mzML files is also supported (see `convert_mzml_to_parquet()`).: "Conversion of mzML files is also supported (see `convert_mzml_to_parquet()`)."
