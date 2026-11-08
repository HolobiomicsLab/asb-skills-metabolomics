---
name: ion-mobility-bin-index-preservation
description: Use when converting profile-mode HDMSE/ion-mobility mass spectrometry
  data (mzML or Waters UNIFI) to a columnar format and the drift-time dimensions (bin,
  dt) must survive the conversion for later IMS trace or CCS analysis.
when_to_use_negative:
- Input is MSE data without ion mobility — bin/dt columns will be absent and IMS-specific
  handling is unnecessary
- A lossy binned summary (e.g. rt/mz-binned aggregation of a few Mb) is sufficient
  and raw drift-time resolution is not needed
- Target format is HDF5 with datasets ms1/ms2/metadata — use arcMS's HDF5 output instead
  of Parquet bin preservation
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: arcMS
  role: R package that converts (HD)MSE mzML/UNIFI data to Parquet, retaining IMS
    bin index and drift time per datapoint; provides convert_bin_to_ccs()
  repo: https://github.com/leesulab/arcMS
- name: Apache Arrow
  role: Columnar Parquet writer/reader used to store and later query the converted
    HDMSE data with bin and dt columns
- name: mzR
  role: Backs mzML parsing inside arcMS's mzML conversion pathway
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:12d84d368b9275dacbd6478a563880d7d1fe9730af7107404d8da0c4a5760102
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Apache Arrow
  - R
  - arcMS
  - mzR
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# ion-mobility-bin-index-preservation

## Summary

Preserve ion-mobility dimensions (drift-time bin index and drift time) when converting Waters (HD)MSE HRMS data from mzML or UNIFI to Parquet, so downstream HDMSE queries, IMS traces, and CCS conversions remain possible.

## When to use

Use when converting profile-mode HDMSE/ion-mobility mass spectrometry data (mzML or Waters UNIFI) to a columnar format and the drift-time dimensions (bin, dt) must survive the conversion for later IMS trace or CCS analysis.

## When NOT to use

- Input is MSE data without ion mobility — bin/dt columns will be absent and IMS-specific handling is unnecessary
- A lossy binned summary (e.g. rt/mz-binned aggregation of a few Mb) is sufficient and raw drift-time resolution is not needed
- Target format is HDF5 with datasets ms1/ms2/metadata — use arcMS's HDF5 output instead of Parquet bin preservation

## Inputs

- Profile-mode mzML file containing IMS/HDMSE data
- Waters UNIFI sample (via API sample_id or analysis_id) with ion mobility (HDMS²E) data
- arcMS R package with mzR-backed parsing and Apache Arrow available

## Outputs

- converted_file.parquet long-format table with columns rt, scanid, mslevel, mz, intensity, bin, dt
- sample_name-metadata. sidecar (metadata also stored inside the Parquet file)
- Optionally CCS values via convert_bin_to_ccs()

## How to apply

During conversion (e.g. with arcMS `convert_mzml_to_parquet` or `convert_one_sample_data`), each scan's spectrum is exploded to a long-format table with one row per m/z datapoint carrying scanid, rt, mslevel, mz, and intensity. Where ion mobility information exists in the mzML, retain the drift-time bin index (bin) and drift time (dt) per datapoint so HDMSE dimensions are preserved. Write with the Apache Arrow/Parquet writer, embedding sample and spectrum metadata (JSON sidecar and Parquet key-value metadata). If absolute CCS values are needed instead of bin indices, use arcMS's `convert_bin_to_ccs()`. Validate by re-opening the Parquet file with `arrow::read_parquet` or `open_dataset` and confirming the expected columns (rt, scanid, mslevel, mz, intensity, bin, dt) are present.

## Related tools

- **arcMS** (R package that converts (HD)MSE mzML/UNIFI data to Parquet, retaining IMS bin index and drift time per datapoint; provides convert_bin_to_ccs()) — https://github.com/leesulab/arcMS
- **Apache Arrow** (Columnar Parquet writer/reader used to store and later query the converted HDMSE data with bin and dt columns)
- **mzR** (Backs mzML parsing inside arcMS's mzML conversion pathway)

## Examples

```
arcms::convert_mzml_to_parquet("sample.mzML"); data <- arrow::open_dataset("converted_file.parquet") |> dplyr::filter(mslevel == "1") |> dplyr::group_by(bin, mz) |> dplyr::summarise(sum_int = sum(intensity))
```

## Evaluation signals

- Re-opened Parquet file loads with arrow::read_parquet or open_dataset and exposes columns rt, scanid, mslevel, mz, intensity, bin, dt
- bin and dt columns are non-empty only for data containing ion mobility (HDMSE), matching the source mzML
- An IMS trace query grouped by (bin, mz) with sum(intensity) succeeds on mslevel == "1"
- convert_bin_to_ccs() returns CCS values from the preserved bin indices
- Metadata JSON sidecar exists alongside the .parquet file and matches the Parquet key-value metadata

## Limitations

- Full profile-mode IMS HDMSE Parquet files are ~200-400 MB; loading the whole sample occupies ~1.9 Gb in memory
- If the source mzML lacks ion mobility information, bin and dt dimensions cannot be reconstructed after conversion
- No changelog is available for the arcMS repository, so version-specific behavior of bin preservation is hard to track

## Evidence

- [abstract] arcMS supports ion mobility dimensions in converted data: "This solution facilitates the manipulation of complex mass spectrometry data, including ion mobility or other additional dimensions"
- [intro] arcMS is compatible with HDMSE and supports mzML conversion: "It is compatible with data containing ion mobility (HDMS<sup>E</sup>) or not (MS<sup>E</sup>). Conversion of mzML files is also supported"
- [intro] Parquet is the minimal-filesize output format: "the [Apache Parquet](https://parquet.apache.org/) format for minimal filesize and fast access"
- [readme] README: mzML conversion entry point and file size claim: "Conversion of mzML files is also supported (see `convert_mzml_to_parquet()`)."
- [readme] README: Parquet files contain low and high energy spectra with metadata stored alongside: [evidence span withheld — anchor status: external_doc]
- [methods] Methods: IMS trace aggregation by bin demonstrates use of preserved bin index: "IMStrace = data_arrow |> to_duckdb() |> filter(mslevel == "1") |> filter(scanid %in% !!scanids) |> group_by(bin, mz)"
- [methods] Methods: bin-to-CCS conversion available: "The `convert_bin_to_ccs()` function can be used as follows"
- [methods] Methods: size of full profile-mode IMS HDMSE Parquet files: "A full data file in profile mode containing IMS, and HDMS<sup>E</sup> data (MS and MS/MS), is around 200-400 Mo."
