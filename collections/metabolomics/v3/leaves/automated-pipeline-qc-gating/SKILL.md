---
name: automated-pipeline-qc-gating
description: Use when RAW LC-MS/MS runs must be flagged or gated automatically before
  database searching based on acquisition-side signals, e.g. an MS2 TIC below a configurable
  percentage of the MS1 TIC.
when_to_use_negative:
- Input RAW files are not from Thermo Fisher instruments — MetaXtract currently supports
  only Thermo RAW files and additional vendor formats are not yet supported
- QC requires identification-based metrics (e.g. peptide/spectrum-level search quality)
  — this skill is identification-free and flags only from acquisition-side TIC signals,
  not database-search outcomes
- Running ms_method/lc_method-based checks on Linux containers — those Thermo method-reading
  calls are not supported on Linux, though scan-header TIC extraction is
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MetaXtract
  role: Extracts MS1/MS2 scan-header Total Ion Current and Scan Start Time (min) directly
    from Thermo RAW files via its YAML-configured CLI or Python API
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Loads the exported MS1 and MS2 scan-header tables, aggregates per-run TIC,
    and computes the MS2/MS1 TIC ratio against the threshold
- name: PyYAML
  role: Parses the YAML config supplying input paths, scan-header column selection,
    and the ms2_tic_min_percent threshold
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:cdc63d77dbe8c674a2de925832404b4fb5c2f2919038c718076dac52dd705e3d
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# automated-pipeline-qc-gating

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identification-free QC gating that extracts scan-level Total Ion Current for MS1 and MS2 from Thermo RAW files via MetaXtract's YAML-driven CLI, computes per-run MS2/MS1 TIC ratios with pandas, and flags/terminates underperforming acquisitions before database searching.

## When to use

Use when RAW LC-MS/MS runs must be flagged or gated automatically before database searching based on acquisition-side signals, e.g. an MS2 TIC below a configurable percentage of the MS1 TIC.

## When NOT to use

- Input RAW files are not from Thermo Fisher instruments — MetaXtract currently supports only Thermo RAW files and additional vendor formats are not yet supported
- QC requires identification-based metrics (e.g. peptide/spectrum-level search quality) — this skill is identification-free and flags only from acquisition-side TIC signals, not database-search outcomes
- Running ms_method/lc_method-based checks on Linux containers — those Thermo method-reading calls are not supported on Linux, though scan-header TIC extraction is

## Inputs

- Thermo Fisher RAW mass spectrometry files (one or more, listed under io.input in the YAML config)
- YAML configuration file enabling scan-header export with Total Ion Current and Scan Start Time (min) columns for MS1 and MS2
- User-configurable MS2 TIC percentage threshold (e.g. ms2_tic_min_percent in the YAML config)

## Outputs

- MS1 and MS2 scan-header tables (CSV/TSV) with Total Ion Current and Scan Start Time (min) columns keyed by scan number
- ms2_tic_flag_report.csv with one row per processed RAW file containing MS2/MS1 TIC ratio, configured threshold, and pass/fail flag status

## How to apply

Run MetaXtract in CLI mode with a YAML config that enables scan-header extraction for both MS levels with 'Total Ion Current: true' under scan_header.MS1 and scan_header.MS2 (plus 'Scan Start Time (min)' for keying). Load the exported MS1 and MS2 scan-header tables with pandas, keyed by scan number and Scan Start Time (min). Aggregate per-run MS1 and MS2 TIC (sum or median over scans) and compute the MS2/MS1 TIC ratio for each processed RAW file. Compare each ratio against the user-configurable YAML percentage threshold (e.g. ms2_tic_min_percent) and write ms2_tic_flag_report.csv with ratio, threshold, and pass/fail flag status, gating the run before any downstream database searching. Rationale: MetaXtract reads RAW files directly via Thermo RawFileReader, so this check is search-free and can run in near real time immediately after acquisition, unlike QC tools such as MSQC that require database search results.

## Related tools

- **MetaXtract** (Extracts MS1/MS2 scan-header Total Ion Current and Scan Start Time (min) directly from Thermo RAW files via its YAML-configured CLI or Python API) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Loads the exported MS1 and MS2 scan-header tables, aggregates per-run TIC, and computes the MS2/MS1 TIC ratio against the threshold)
- **PyYAML** (Parses the YAML config supplying input paths, scan-header column selection, and the ms2_tic_min_percent threshold)

## Examples

```
python main.py --config /path/to/config.yml
```

## Evaluation signals

- ms2_tic_flag_report.csv exists and parses as valid CSV with ratio and flag columns
- All computed MS2/MS1 TIC ratio values fall within [0, 1]
- Every flag status is exactly (ratio < configured ms2_tic_min_percent threshold) — no run is flagged or passed inconsistently
- Report contains exactly one row per processed RAW file listed in io.input
- Exported scan-header tables contain the Total Ion Current column for both MS1 and MS2 (PSI-MS term MS:1000285) with Scan Start Time (min) for scan-level keying

## Limitations

- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported
- Linking QC deviations to instrument-level causes is described as plausible rather than validated — the flag is a gate, not a diagnosis
- No version number or changelog is stated for MetaXtract, and no default ms2_tic_min_percent threshold is documented; the cutoff must be supplied by the user

## Evidence

- [abstract] MetaXtract enables search-free, near real-time performance monitoring by relying on acquisition-side signals, providing actionable indicators immediately after data acquisition: "MetaXtract enables search-free, near real-time performance monitoring by relying on acquisition-side signals, providing actionable indicators immediately after data acquisition"
- [readme] MetaXtract can be fully configured using a YAML configuration file.: "MetaXtract can be fully configured using a YAML configuration file.  
This allows reproducible, automated runs without passing long CLI arguments."
- [intro] MetaXtract is a lightweight Python-based tool that directly reads RAW files using the Thermo Fisher RAWFileReader library: "we developed MetaXtract, a lightweight Python-based tool that directly reads RAW files using the Thermo Fisher RAWFileReader library"
- [readme] Total Ion Current column mapping to PSI-MS MS:1000285: "`Total Ion Current` | `total ion current`, `MS:1000285` | Direct match."
