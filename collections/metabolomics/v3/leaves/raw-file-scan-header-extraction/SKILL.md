---
name: raw-file-scan-header-extraction
description: Use when you have Thermo .RAW mass spectrometry files and need structured
  scan-header tables (TIC, Scan Start Time, Base Peak Intensity) for search-free QC,
  method troubleshooting, FAIR metadata export, or machine-learning feature extraction.
when_to_use_negative:
- Input files are not Thermo RAW format — MetaXtract currently supports only Thermo
  Fisher RAW files; mzML or other vendor formats are not supported.
- You need database-search-dependent QC metrics — MetaXtract deliberately relies only
  on acquisition-side signals and does not compute identification-based metrics.
- Running inside a Linux container and you require MS-method or LC-method extraction
  — these Thermo method-reading calls are not supported on Linux.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaXtract
  role: Primary tool — reads Thermo RAW files via RawFileReader and exports structured
    scan-header tables in CLI, GUI, or Python-library mode
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:28e06483ef654fac15d29002be44b62403d1ae1085ee06b2e28137ff89089a9f
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
  - PyYAML
  - pandas
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# raw-file-scan-header-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract scan-level headers (retention time, total ion current, ion injection time, peak counts, base peak m/z) from MS1 and MS2 scans of Thermo Fisher RAW files using MetaXtract, exporting them as structured CSV/TSV tables that enable identification-free QC and downstream computational analysis.

## When to use

Use when you have Thermo .RAW mass spectrometry files and need structured scan-header tables (TIC, Scan Start Time, Base Peak Intensity) for search-free QC, method troubleshooting, FAIR metadata export, or machine-learning feature extraction.

## When NOT to use

- Input files are not Thermo RAW format — MetaXtract currently supports only Thermo Fisher RAW files; mzML or other vendor formats are not supported.
- You need database-search-dependent QC metrics — MetaXtract deliberately relies only on acquisition-side signals and does not compute identification-based metrics.
- Running inside a Linux container and you require MS-method or LC-method extraction — these Thermo method-reading calls are not supported on Linux.

## Inputs

- Thermo Fisher .RAW mass spectrometry files (one or many)
- YAML configuration file specifying scan_header columns for MS1/MS2

## Outputs

- Per-RAW MS1/MS2 scan-header CSV/TSV tables keyed by scan number and Scan Start Time (min)
- Optional file-based details TSV (instrument details, scan counts, run statistics, sample information)
- Optional SDRF-Proteomics metadata.sdrf.tsv

## How to apply

Run MetaXtract via CLI with a YAML config selecting the scan_header section for MS1 and/or MS2, toggling specific columns such as 'Total Ion Current', 'Scan Start Time (min)', 'Ion Injection Time (ms)', 'Base Peak Intensity', 'Base Peak m/z', and 'Scan Mode' (or set select_all for the complete scan header). MetaXtract reads the RAW file through Thermo RawFileReader and writes one CSV per scan level (e.g. *_technical_details_ms2_*.csv via GetMoreMSInfos), using PSI-MS/mzML-style column names (e.g. MS:1000285 for Total Ion Current, MS:1000016 for Scan Start Time). Decide whether to enable ms_method/lc_method exports — these Thermo method-reading calls are unsupported on Linux containers and must be set to false there. Outputs land in a per-RAW subfolder under output_dir, optionally alongside Parquet peak lists, SDRF metadata, and Plotly HTML visualizations.

## Related tools

- **MetaXtract** (Primary tool — reads Thermo RAW files via RawFileReader and exports structured scan-header tables in CLI, GUI, or Python-library mode) — https://github.com/Rappsilber-Laboratory/MetaXtract

## Examples

```
python main.py --config /path/to/config.yml
```

## Evaluation signals

- Output CSV columns follow PSI-MS/mzML-style names (e.g. 'Total Ion Current', 'Scan Start Time (min)') with thermo_ prefixes only for trailer fields lacking a PSI-MS match
- Each exported scan-header table contains a row per scan with a parseable numeric 'Total Ion Current' and 'Scan Start Time (min)'
- MS1 and MS2 tables are distinguishable and internally consistent with the RAW file's scan counts reported in the file-based details TSV
- Running with select_all: true for a level yields a superset of columns versus selective column toggling
- On Linux containers, ms_method and lc_method outputs must be absent or explicitly disabled, matching the config

## Limitations

- Currently supports only Thermo Fisher RAW files; additional vendor file formats not yet supported
- MS-method and LC-method export options are not supported on Linux containers
- MetaXtract is not yet integrated as a Proteome Discoverer node (planned for future versions)

## Evidence
