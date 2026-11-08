---
name: raw-file-runtime-measurement
description: Use when benchmarking MetaXtract's per-RAW-file wall-clock runtime in
  the PRIDE Snakemake workflow, to reproduce or validate the ~40 s/file reference
  performance before scaling to large batches.
when_to_use_negative:
- Input RAW files are not Thermo Fisher format — MetaXtract currently supports only
  Thermo RAW files and additional vendor formats are not yet supported
- The goal is QC metric extraction or spectral peak-list export rather than runtime
  benchmarking — use the metadata-extraction workflow outputs directly instead
- Running on Linux containers where MS-method and LC-method export is disabled — runtimes
  will not be comparable to the Windows 11 reference configuration
tools:
- name: MetaXtract
  role: Tool under benchmark; reads RAW files via RawFileReader and performs the metadata
    annotation whose per-file wall-clock runtime is being measured
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Parses the runtime_summary_*.tsv logs, extracts per-file runtimes, and computes
    mean/min/max summary statistics
- name: Snakemake
  role: Orchestrates the PRIDE RAW-file download-by-year/count and end-to-end metadata
    annotation runs being timed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:fb049a3205c87aac492c82fdb6b9b3c4d3f5fbc23bfe28392f3d00a86a7eb828
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Rappsilber-Laboratory/MetaXtract
  repo_url: https://github.com/Rappsilber-Laboratory/MetaXtract
  tools:
  - MetaXtract
  - Snakemake
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# raw-file-runtime-measurement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the wall-clock runtime MetaXtract needs per Thermo RAW file when running the PRIDE download-and-annotate Snakemake workflow, validating the paper's reported ~40 s per file on a Windows 11 reference workstation.

## When to use

Use when benchmarking MetaXtract's per-RAW-file wall-clock runtime in the PRIDE Snakemake workflow, to reproduce or validate the ~40 s/file reference performance before scaling to large batches.

## When NOT to use

- Input RAW files are not Thermo Fisher format — MetaXtract currently supports only Thermo RAW files and additional vendor formats are not yet supported
- The goal is QC metric extraction or spectral peak-list export rather than runtime benchmarking — use the metadata-extraction workflow outputs directly instead
- Running on Linux containers where MS-method and LC-method export is disabled — runtimes will not be comparable to the Windows 11 reference configuration

## Inputs

- Thermo Fisher .RAW files downloaded from PRIDE (e.g. the last 20 RAW files submitted in January 2025)
- runtime_summary_*.tsv files written by MetaXtract per processed RAW file
- MetaXtract YAML config (config.yml) specifying outputs such as ms2_peaklist_export, ms1_peaklist_export, and visualisation flags

## Outputs

- runtime_comparison.tsv with per-file runtime, deviation from the ~40 s reference, and per-file status
- Summary statistics (mean, minimum, maximum) of per-file runtimes across the batch

## How to apply

Run the MetaXtract Snakemake workflow from the GitHub repository, which automatically downloads RAW files from PRIDE (by year and file count — e.g. the last 20 RAW files submitted in January 2025) and runs end-to-end metadata annotation into structured TSV/CSV. Parse the runtime_summary_*.tsv files written by MetaXtract for each processed RAW file using pandas, and extract the per-file runtime in seconds. Compute summary statistics (mean, minimum, maximum) across the 20 files and compare each file's runtime against the reported reference average of ~40 s per file recorded on the Windows 11 reference laptop (Intel Core i9-13900HX, 32 GB RAM, NVIDIA RTX 4070). Tabulate per-file runtime, deviation from the 40 s reference, and per-file status, and save the comparison as runtime_comparison.tsv. Interpret deviations in light of hardware and export options, since peak-list/Parquet exports and visualisation flags change the work done per file.

## Related tools

- **MetaXtract** (Tool under benchmark; reads RAW files via RawFileReader and performs the metadata annotation whose per-file wall-clock runtime is being measured) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Parses the runtime_summary_*.tsv logs, extracts per-file runtimes, and computes mean/min/max summary statistics)
- **Snakemake** (Orchestrates the PRIDE RAW-file download-by-year/count and end-to-end metadata annotation runs being timed)

## Examples

```
import pandas as pd; df = pd.read_csv('runtime_summary.tsv', sep='\t'); df['runtime_s'].agg(['mean','min','max'])
```

## Evaluation signals

- runtime_comparison.tsv exists and is a well-formed TSV with one row per processed RAW file
- Each row's runtime value is a positive number
- Mean runtime across the batch is in the vicinity of the ~40 s/file reference on the Windows 11 i9-13900HX/32 GB/RTX 4070 hardware
- Per-file deviation from the 40 s reference is populated and finite for every row
- Number of rows matches the number of RAW files requested from PRIDE (e.g. 20 files for the January 2025 batch)

## Limitations

- The ~40 s/file figure is tied to a specific reference machine (Windows 11, Intel Core i9-13900HX, 32 GB RAM, NVIDIA RTX 4070); other hardware will deviate
- MetaXtract currently supports only Thermo Fisher RAW files; other vendor formats cannot be benchmarked
- On Linux containers, MS-method and LC-method extraction is not supported, changing the workload per file versus the Windows reference
- No version number is stated for MetaXtract, so runtime comparisons across releases are hard to pin down

## Evidence
