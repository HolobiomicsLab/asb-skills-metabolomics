---
name: mslevel-collision-energy-annotation
description: Use when converting Waters UNIFI (HD)MSE or mzML profile-mode data to
  Parquet/HDF5 with arcMS and you need to preserve and exploit the low vs high collision
  energy (MS1/MS2) distinction for filtering and chromatogram/spectrum queries.
when_to_use_negative:
- Input data are centroided feature tables or peak lists where MS level is already
  resolved — arcMS conversion operates on profile-mode mzML or raw UNIFI data
- The target format is HDF5 and you only need one energy level — reading the ms1 or
  ms2 dataset directly is simpler than filtering a combined table
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: arcMS
  role: R package that converts (HD)MSE/mzML data to Parquet, annotating each datapoint
    with mslevel (low vs high collision energy) in the long-format output
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Used to open the Parquet file and filter on mslevel (e.g. filter(mslevel ==
    "1")) before group_by(rt)/summarise TIC aggregation
  repo: https://arrow.apache.org/
- name: rhdf5
  role: Reads the ms1 and ms2 HDF5 datasets, which store low- and high-collision-energy
    spectra separately instead of via an mslevel column
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:84be10d8bf350a36d6eb21734d3fb0991bfcaedd6149f56a45a97deec1987e33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - R
  - arcMS
  - mzR
  - rhdf5
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# mslevel-collision-energy-annotation

## Summary

Annotates each spectral datapoint in converted HRMS data with an mslevel discriminator separating low collision energy (MS1) from high collision energy (MS2) scans, so downstream Arrow/dplyr filtering (e.g. TIC, EIC) can operate on the correct energy level in Parquet files produced by arcMS.

## When to use

Use when converting Waters UNIFI (HD)MSE or mzML profile-mode data to Parquet/HDF5 with arcMS and you need to preserve and exploit the low vs high collision energy (MS1/MS2) distinction for filtering and chromatogram/spectrum queries.

## When NOT to use

- Input data are centroided feature tables or peak lists where MS level is already resolved — arcMS conversion operates on profile-mode mzML or raw UNIFI data
- The target format is HDF5 and you only need one energy level — reading the ms1 or ms2 dataset directly is simpler than filtering a combined table

## Inputs

- Profile-mode mzML file (via convert_mzml_to_parquet with chunksize argument)
- Waters UNIFI (HD)MSE sample data accessed through the UNIFI Web API (sample_id / analysis_id)

## Outputs

- converted_file.parquet with long-format table exposing columns rt, scanid, mslevel, mz, intensity (plus bin, dt for IMS/HDMSE data)
- HDF5 file with ms1 (low energy) and ms2 (high energy) datasets plus metadata datasets

## How to apply

During conversion (convert_mzml_to_parquet or the UNIFI API pathway), each scan's spectrum is exploded to a long-format table with one row per m/z datapoint carrying scanid, rt, mslevel, mz and intensity; mslevel encodes low vs high collision energy (MS1 vs MS2) as acquired in HDMSE. In Parquet output, both low and high energy spectra live in the single converted_file.parquet table, and the mslevel column is the sole discriminator, so filters like filter(mslevel == "1") isolate MS1 for TIC computation. In HDF5 output the annotation is instead realized structurally: low energy is stored in the 'ms1' dataset and high energy in 'ms2'. Validate by grouping on rt after an mslevel filter (group_by(rt) |> summarise(sum(intensity))) to confirm the TIC aggregation runs on the intended energy level.

## Related tools

- **arcMS** (R package that converts (HD)MSE/mzML data to Parquet, annotating each datapoint with mslevel (low vs high collision energy) in the long-format output) — https://github.com/leesulab/arcMS
- **Arrow** (Used to open the Parquet file and filter on mslevel (e.g. filter(mslevel == "1")) before group_by(rt)/summarise TIC aggregation) — https://arrow.apache.org/
- **rhdf5** (Reads the ms1 and ms2 HDF5 datasets, which store low- and high-collision-energy spectra separately instead of via an mslevel column)

## Examples

```
TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- Re-open converted_file.parquet with arrow::read_parquet or open_dataset and confirm mslevel is an expected column alongside rt, scanid, mz, intensity, bin, dt
- filter(mslevel == "1") followed by group_by(rt) |> summarise(sum(intensity)) TIC aggregation succeeds and returns per-rt intensity sums (~64 Kb result for the example query)
- For HDF5 output, rhdf5::h5read returns distinct 'ms1' (low energy) and 'ms2' (high energy) datasets
- An EIC filter (mz window e.g. 216.0016–216.2016) grouped by rt and scanid returns scanids consistent with the annotated mslevel

## Limitations

- Parquet stores both low and high energy spectra in a single table, so mslevel must be filtered explicitly; HDF5 separates them into ms1/ms2 datasets instead
- Full profile-mode IMS HDMSE files are ~200–400 MB as Parquet; pure Arrow binned rt/mz aggregation queries can take ~30 s to 1 min, though DuckDB (to_duckdb) runs in a few seconds

## Evidence

- [readme] Parquet files contain both low and high energy spectra (HDMSe), and HDF5 files contain low energy in the “ms1” dataset, high energy in the “ms2” dataset: [evidence span withheld — anchor status: external_doc]
- [methods] TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)): "TIC = data_arrow |>
  filter(mslevel == "1") |>
  group_by(rt) |> 
  summarise(int = sum(intensity)) |> 
  collect()"
- [intro] It is compatible with data containing ion mobility (HDMSE) or not (MSE). Conversion of mzML files is also supported: "It is compatible with data containing ion mobility
(HDMS<sup>E</sup>) or not (MS<sup>E</sup>). Conversion of mzML files is
also supported"
