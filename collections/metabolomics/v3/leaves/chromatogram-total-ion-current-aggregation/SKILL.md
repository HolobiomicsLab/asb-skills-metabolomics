---
name: chromatogram-total-ion-current-aggregation
description: Use when you have a columnar Parquet HRMS file with rt, intensity and
  mslevel columns and need a TIC (intensity summed per retention time for MS1 scans)
  without loading the whole file into memory.
when_to_use_negative:
- When you need individual scan-level or per-m/z intensities rather than a per-retention-time
  intensity sum (use EIC/spectrum filter queries instead)
- When the Parquet file is small enough to fit comfortably in RAM and read_parquet()
  plus in-memory dplyr is simpler
- When the input is HDF5 (sample.h5 with ms1/ms2 datasets) rather than Parquet — the
  Arrow/dplyr on-disk pipeline applies to Parquet files
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: arcMS
  role: Produces the Parquet-format HRMS files that this TIC aggregation query operates
    on
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: open_dataset() opens the Parquet file on-disk so the TIC query filters/groups/summarises
    without loading the full ~1.9 Gb dataset
- name: dplyr
  role: Provides the filter(mslevel == "1"), group_by(rt), summarise(int = sum(intensity))
    and collect() verbs executed on the Arrow dataset
- name: R
  role: Language in which the arcMS package and the Arrow/dplyr TIC pipeline are implemented
  repo: https://github.com/leesulab/arcMS
- name: arcms-dataviz
  role: Shiny companion app that retrieves chromatograms and MS spectra from Parquet
    with Apache Arrow as a reference implementation of this skill
  repo: https://github.com/leesulab/arcms-dataviz
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:2af51d0d0766d62a082ad397ae7eb4784ce357d41acdedaa2213f2f3ff723251
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - R
  - arcMS
  - arcms-dataviz
  - dplyr
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# chromatogram-total-ion-current-aggregation

## Summary

Computes a total ion chromatogram (TIC) from an HRMS Parquet file (e.g. arcMS-converted Waters UNIFI (HD)MS^E data) using an on-disk Apache Arrow/dplyr query, so only the ~64 Kb aggregated result is collected into RAM instead of the ~1.9 Gb full dataset.

## When to use

Use when you have a columnar Parquet HRMS file with rt, intensity and mslevel columns and need a TIC (intensity summed per retention time for MS1 scans) without loading the whole file into memory.

## When NOT to use

- When you need individual scan-level or per-m/z intensities rather than a per-retention-time intensity sum (use EIC/spectrum filter queries instead)
- When the Parquet file is small enough to fit comfortably in RAM and read_parquet() plus in-memory dplyr is simpler
- When the input is HDF5 (sample.h5 with ms1/ms2 datasets) rather than Parquet — the Arrow/dplyr on-disk pipeline applies to Parquet files

## Inputs

- HRMS data in Apache Parquet columnar format (e.g. arcMS output such as cal1.parquet converted from Waters UNIFI (HD)MS^E data)
- Columns: rt (retention time), intensity, mslevel (with "1" denoting low collision energy MS1 scans); optionally scanid, mz, bin for related queries

## Outputs

- Two-column R data frame (rt, int) of ~64 Kb containing summed low-energy intensities across the full retention-time range (the TIC)

## How to apply

Open the Parquet file with arrow::open_dataset() to create a FileSystemDataset pointer, avoiding loading the ~1.9 Gb of data into RAM. Build a dplyr pipeline on the Arrow dataset: filter(mslevel == "1") to keep only low collision energy (MS1) scans, group_by(rt), and summarise(int = sum(intensity)). Finish with collect() to pull only the aggregated result (~64 Kb, computed in around 1s per the paper) into an R data frame. For heavier aggregations (e.g. binned rt/mz 2D maps), route the query through DuckDB via to_duckdb(), which runs in a few seconds versus ~30s-1 min for the pure Arrow binned query.

## Related tools

- **arcMS** (Produces the Parquet-format HRMS files that this TIC aggregation query operates on) — https://github.com/leesulab/arcMS
- **Arrow** (open_dataset() opens the Parquet file on-disk so the TIC query filters/groups/summarises without loading the full ~1.9 Gb dataset)
- **dplyr** (Provides the filter(mslevel == "1"), group_by(rt), summarise(int = sum(intensity)) and collect() verbs executed on the Arrow dataset)
- **R** (Language in which the arcMS package and the Arrow/dplyr TIC pipeline are implemented) — https://github.com/leesulab/arcMS
- **arcms-dataviz** (Shiny companion app that retrieves chromatograms and MS spectra from Parquet with Apache Arrow as a reference implementation of this skill) — https://github.com/leesulab/arcms-dataviz

## Examples

```
TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()
```

## Evaluation signals

- Output data frame has exactly two columns named rt and int
- Resulting table size is on the order of tens of Kb (~64 Kb for the cal1 example), not the full ~1.9 Gb dataset
- int values are positive sums of low-energy (mslevel == "1") scan intensities across the full retention-time range
- Query completes in roughly 1s when run on-disk via arrow::open_dataset() as reported in the paper
- The R session never holds the full Parquet data in memory — only the collected aggregate

## Limitations

- The pure Arrow binned rt/mz aggregation variant is slower (~30s to 1 min); routing through DuckDB via to_duckdb() runs in a few seconds
- Only low-energy (mslevel == "1") scans are included; high-energy MS/MS scans are filtered out by design
- Example data occupies ~1.9 Gb when loaded, so collecting unfiltered data defeats the purpose of the on-disk query

## Evidence

- [methods] TIC pipeline on Arrow dataset: "TIC = data_arrow |> filter(mslevel == "1") |> group_by(rt) |> summarise(int = sum(intensity)) |> collect()"
- [methods] Aggregate result size and timing: "This took around 1s and the resulting data is only 64 Kb!"
- [methods] Full dataset memory footprint: "the loaded data for this example sample occupies \~1.9 Gb in memory"
- [readme] On-disk Arrow retrieval in the dataviz app: [evidence span withheld — anchor status: not_found]
