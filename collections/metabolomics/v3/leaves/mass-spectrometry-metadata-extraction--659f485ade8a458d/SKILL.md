---
name: mass-spectrometry-metadata-extraction
description: Use when Thermo RAW files (e.g. downloaded from PRIDE accessions) need
  instrument-, run-, and scan-level metadata extracted into structured TSV/CSV tables
  for QC, repository submission, or batch annotation workflows.
when_to_use_negative:
- Input RAW files are from vendors other than Thermo Fisher — MetaXtract currently
  supports only Thermo RAW files
- A database-search-dependent QC metric is required (MetaXtract relies on acquisition-side
  signals, not search results)
- The goal is LC-MS/MS method extraction inside a Linux container — ms_method/lc_method
  Thermo calls are unsupported on Linux
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaXtract
  role: Primary tool that parses Thermo RAW files via RawFileReader (pythonnet/Mono),
    extracts file-, instrument-, run-, and scan-level metadata, and exports TSV/CSV/Parquet
    and SDRF outputs
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: Snakemake workflow (bundled in MetaXtract repo)
  role: Automates downloading RAW files from PRIDE by year and file count and running
    MetaXtract's end-to-end annotation on each file
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:6aa70f7674c15e4a7e42f344f0fdbade47a18ca0c6f011b5a59e628da5ada24e
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
  - Snakemake workflow (bundled in MetaXtract repo)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mass-spectrometry-metadata-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts comprehensive acquisition metadata (instrument details, MS1/MS2 scan headers, technical trailer details, peak lists) directly from Thermo Fisher RAW files using MetaXtract, exporting structured TSV/CSV and Parquet tables for FAIR data practices, QC, and automated pipeline annotation.

## When to use

Use when Thermo RAW files (e.g. downloaded from PRIDE accessions) need instrument-, run-, and scan-level metadata extracted into structured TSV/CSV tables for QC, repository submission, or batch annotation workflows.

## When NOT to use

- Input RAW files are from vendors other than Thermo Fisher — MetaXtract currently supports only Thermo RAW files
- A database-search-dependent QC metric is required (MetaXtract relies on acquisition-side signals, not search results)
- The goal is LC-MS/MS method extraction inside a Linux container — ms_method/lc_method Thermo calls are unsupported on Linux

## Inputs

- Thermo Fisher RAW mass spectrometry files (e.g. downloaded from PRIDE accessions such as PXD060351)
- YAML configuration file (config.yml) specifying io.input, outputs, scan_header, and visualisation options
- Optional user-filled SDRF metadata TSV for sdrf output completion

## Outputs

- File-based details TSV (instrument details, scan counts, run statistics, sample information)
- MS1 and MS2 scan-header TSV/CSV tables with PSI-MS/mzML-style column names
- MS2 technical details CSV via GetMoreMSInfos (`*_technical_details_ms2_*.csv`)
- MS1/MS2 peak lists in Parquet format
- Optional metadata.sdrf.tsv (SDRF-Proteomics) and interactive Plotly HTML reports

## How to apply

Run MetaXtract via its CLI or Linux container against one or more Thermo .RAW files, either by passing a YAML config (`python main.py --config /path/to/config.yml`) or CLI flags. Enable `file_based_details` to write a TSV with instrument details, scan counts, run statistics, and sample information; enable `select_all` under `scan_header` for MS1 and MS2 to export complete scan-header tables with PSI-MS/mzML-named columns (e.g. `Scan Start Time (min)`, `Total Ion Current`, `Ion Injection Time (ms)`, `Base Peak m/z`). Optionally enable `ms1_peaklist_export`/`ms2_peaklist_export` for Parquet peak lists. Keep `ms_method` and `lc_method` disabled in Linux container mode because the Thermo method-reading calls are unsupported on Linux. Write outputs to one subfolder per RAW file under a shared output directory; in the paper's Snakemake workflow this is applied end-to-end to RAW files downloaded from PRIDE (by year and file count), e.g. the last 20 files submitted in January 2025 for PXD060351, PXD060271, and PXD060112.

## Related tools

- **MetaXtract** (Primary tool that parses Thermo RAW files via RawFileReader (pythonnet/Mono), extracts file-, instrument-, run-, and scan-level metadata, and exports TSV/CSV/Parquet and SDRF outputs) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **Snakemake workflow (bundled in MetaXtract repo)** (Automates downloading RAW files from PRIDE by year and file count and running MetaXtract's end-to-end annotation on each file) — https://github.com/Rappsilber-Laboratory/MetaXtract

## Examples

```
python main.py --config /path/to/config.yml
```

## Evaluation signals

- For each RAW file, the expected file-based details TSV exists under its per-file output subfolder and is non-empty
- MS1 and MS2 scan-header TSV/CSV tables exist per file and contain PSI-MS-style columns (e.g. 'Scan Start Time (min)', 'Total Ion Current', 'Base Peak m/z')
- MS-method/LC-method outputs are absent (or unchanged) when running in Linux container mode, matching the disabled ms_method/lc_method flags
- Scan-header tables row counts are consistent with the MS1/MS2 scan counts reported in the file-based details TSV
- In the PRIDE batch use case, all three accessions (PXD060351, PXD060271, PXD060112) yield per-submission structured TSV/CSV metadata outputs

## Limitations

- Currently supports only Thermo Fisher RAW files; additional vendor formats are not yet supported
- MS-method and LC-method extraction is not supported on Linux containers
- No version number or changelog is stated for MetaXtract
- MetaXtract is not yet integrated as a Proteome Discoverer node (planned for future versions)

## Evidence
