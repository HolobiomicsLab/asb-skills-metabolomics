---
name: on-disk-dplyr-aggregation-of-spectra
description: Use when HRMS data is stored as Parquet (~200-400 MB profile-mode files)
  and you need filtered, aggregated chromatograms (TIC/BPI/EIC) or spectra, but the
  full file must not be loaded into memory.
when_to_use_negative:
- The data is already a small feature table or in-memory data frame — the open_dataset/to_duckdb
  streaming machinery adds overhead with no benefit.
- The file is HDF5 rather than Parquet; this skill targets Arrow-compatible columnar
  Parquet data, not rhdf5 'ms1'/'ms2' datasets.
- You need raw scan-level profile points rather than filtered/aggregated chromatograms
  or spectra — collect() on unaggregated data would pull ~1.9 Gb into memory.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3077
tools:
- name: arcMS
  role: Produces the Parquet columnar HRMS files (and EIC/TIC visualizations via its
    Shiny app) that this out-of-core aggregation skill queries
  repo: https://github.com/leesulab/arcMS
- name: Arrow
  role: Opens the Parquet dataset on disk with open_dataset()/read_parquet and supports
    dplyr filter/group_by/summarise lazily before collect()
- name: DuckDB
  role: Receives the Arrow dataset via zero-copy to_duckdb() to provide the SQL-backed
    query engine that runs binned aggregations in a few seconds instead of ~30s-1
    min
- name: dplyr
  role: Expresses the filter (mslevel, mz window), mutate (rt/mz binning), group_by
    and summarise(sum(intensity)) aggregation steps
- name: Shiny
  role: Companion arcms-dataviz app that applies the same Arrow-based out-of-core
    retrieval to visualize TIC, BPI and EIC chromatograms from Parquet
  repo: https://github.com/leesulab/arcms-dataviz
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae160
    title: 'arcMS: transformation of multi-dimensional high-resolution mass spectrometry
      data to columnar format for compact storage and fast access'
  source_content_hash: sha256:bff55fc9bee7c84f2a7d288b8603a5c46571561ae58c80e091fca26c8460fd2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leesulab/arcMS
  tools:
  - Arrow
  - DuckDB
  - R
  - Shiny
  - arcMS
  - dplyr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# on-disk dplyr aggregation of spectra

## Summary

Query Parquet-stored HRMS profile data out-of-core with Arrow plus a zero-copy DuckDB stream, using dplyr filter/group_by/summarise to aggregate chromatograms or spectra (e.g. an EIC summed over 216.0016 < mz < 216.2016) without loading the ~200-400 MB file into RAM.

## When to use

Use when HRMS data is stored as Parquet (~200-400 MB profile-mode files) and you need filtered, aggregated chromatograms (TIC/BPI/EIC) or spectra, but the full file must not be loaded into memory.

## When NOT to use

- The data is already a small feature table or in-memory data frame — the open_dataset/to_duckdb streaming machinery adds overhead with no benefit.
- The file is HDF5 rather than Parquet; this skill targets Arrow-compatible columnar Parquet data, not rhdf5 'ms1'/'ms2' datasets.
- You need raw scan-level profile points rather than filtered/aggregated chromatograms or spectra — collect() on unaggregated data would pull ~1.9 Gb into memory.

## Inputs

- Apache Parquet file of HRMS data converted with arcMS (columns: rt, mz, intensity, scanid, mslevel, bin), e.g. cal1-full-long.parquet (~200-400 MB profile mode)
- m/z target and window half-width for an EIC (e.g. 216.1016 +/- 0.1)

## Outputs

- Small aggregated R data frame / tibble, e.g. EIC table with rt and intensity columns (exportable as eic_atrazine.csv)
- Optionally binned rt/mz aggregation tables or MS/IMS traces (several Mb)

## How to apply

Open the Parquet file on disk with arrow::open_dataset() so the ~200-400 MB profile-mode HDMSE file is never read into RAM, then stream it into DuckDB with to_duckdb() for a zero-copy SQL-capable interface. Filter with dplyr syntax: first restrict to low-collision-energy data (mslevel == "1"), then window m/z values, e.g. 216.0016 < mz < 216.2016 (+/-0.1 around the atrazine m/z of 216.1016). Group by rt (and scanid for an EIC) and aggregate with summarise(intensity = sum(intensity)); for binned 2D aggregation, mutate rt_binned = floor(rt/0.1)*0.1 and mz_binned = floor(mz/1)*1 first — pure Arrow handles this in ~30s to 1 min, while DuckDB runs in a few seconds. Call collect() only at the end so just the small aggregated table (~64 Kb for a TIC) enters memory, then arrange by retention time.

## Related tools

- **arcMS** (Produces the Parquet columnar HRMS files (and EIC/TIC visualizations via its Shiny app) that this out-of-core aggregation skill queries) — https://github.com/leesulab/arcMS
- **Arrow** (Opens the Parquet dataset on disk with open_dataset()/read_parquet and supports dplyr filter/group_by/summarise lazily before collect())
- **DuckDB** (Receives the Arrow dataset via zero-copy to_duckdb() to provide the SQL-backed query engine that runs binned aggregations in a few seconds instead of ~30s-1 min)
- **dplyr** (Expresses the filter (mslevel, mz window), mutate (rt/mz binning), group_by and summarise(sum(intensity)) aggregation steps)
- **Shiny** (Companion arcms-dataviz app that applies the same Arrow-based out-of-core retrieval to visualize TIC, BPI and EIC chromatograms from Parquet) — https://github.com/leesulab/arcms-dataviz

## Examples

```
eic = data_arrow |> to_duckdb() |> filter(mslevel == "1") |> filter(mz > 216.0016 & mz < 216.2016) |> group_by(rt, scanid) |> summarise(intensity = sum(intensity)) |> collect()
```

## Evaluation signals

- The resulting table is small (e.g. a TIC aggregation yields only ~64 Kb), confirming aggregation happened on disk before collect()
- eic_atrazine.csv exists and contains exactly the rt and intensity columns with positive summed intensities
- EIC rows are ordered by increasing retention time after arrange(rt)
- The binned query with to_duckdb() completes in a few seconds versus ~30s to 1 min for pure Arrow, matching the reported DuckDB speedup
- Peak memory stays well below the ~1.9 Gb the fully loaded sample occupies, demonstrating out-of-core behavior

## Limitations

- Pure Arrow aggregation of binned rt/mz queries takes ~30s to 1 min; DuckDB is needed to run the same query in a few seconds.
- EIC m/z windows must be expressed against the profile-mode point table (scanid-level grouping); data must have been converted with arcMS to Parquet, and ion-mobility dimensions require the bin column conventions of HDMS'E' files.
- Out-of-core behavior only holds until collect(); calling collect() before summarise would load ~1.9 Gb of profile data into RAM.

## Evidence

- [methods] filter(mslevel == "1") then mz window and summarise: "filter(mz > 216.0016 & mz < 216.2016) |> 
  group_by(rt, scanid) |>
  summarise(intensity = sum(intensity))"
- [methods] DuckDB speed vs Arrow: [evidence span withheld — anchor status: not_found]
- [methods] TIC aggregation output size: "This took around 1s and the resulting data is only 64 Kb!"
- [readme] out-of-core retrieval in dataviz app: "Data is not fully loaded in RAM but chromatograms and MS spectra are
quickly retrieved from the parquet file with the [Apache Arrow
library]"
- [methods] full-file memory footprint: "the loaded data for this example sample occupies \~1.9 Gb in memory"
