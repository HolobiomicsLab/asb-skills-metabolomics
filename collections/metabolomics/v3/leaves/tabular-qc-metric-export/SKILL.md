---
name: tabular-qc-metric-export
description: Use when Thermo RAW-file acquisition performance must be assessed without
  database searching — e.g. diagnosing peptide-identification loss in HeLa Pierce
  digest QC runs by comparing MS1/MS2 TIC medians between underperforming and reference
  runs.
when_to_use_negative:
- The RAW files are from a non-Thermo vendor — MetaXtract currently supports only
  Thermo Fisher RAW files
- QC requires peptide-level or identification-based metrics — this skill is search-free
  and uses only acquisition-side signals like TIC
- You need full spectral peak arrays (m/z, intensity, resolution, noise, charge state)
  for modelling rather than scan-level summary metrics — use MetaXtract's peak-list
  (Parquet) export instead
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: Extracts MS1 and MS2 scan-header tables (including 'Total Ion Current' per
    scan) directly from Thermo RAW files without database searching
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Computes per-run median TIC values and the UP-vs-reference ratio/fold-reduction
    metrics, then writes the summary CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:47d9705d392d4b37c3d4cc115d9e205c41b46c1f91b7030a8e44b06de63ff95e
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
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# tabular-qc-metric-export

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes search-free QC metrics from MetaXtract-extracted Thermo RAW-file scan headers, comparing MS1 and MS2 'Total Ion Current' medians between underperforming and reference QC runs to localize signal loss (e.g. 8.4% MS1 drop vs 9.2-fold MS2 reduction pointing to isolation/fragment transmission).

## When to use

Use when Thermo RAW-file acquisition performance must be assessed without database searching — e.g. diagnosing peptide-identification loss in HeLa Pierce digest QC runs by comparing MS1/MS2 TIC medians between underperforming and reference runs.

## When NOT to use

- The RAW files are from a non-Thermo vendor — MetaXtract currently supports only Thermo Fisher RAW files
- QC requires peptide-level or identification-based metrics — this skill is search-free and uses only acquisition-side signals like TIC
- You need full spectral peak arrays (m/z, intensity, resolution, noise, charge state) for modelling rather than scan-level summary metrics — use MetaXtract's peak-list (Parquet) export instead

## Inputs

- Thermo Fisher RAW files from QC runs (e.g. four HeLa Pierce Protein Digest Standard runs: two underperforming, two reference)
- MetaXtract-exported MS1 scan-header table containing the 'Total Ion Current' column per scan (CSV/TSV)
- MetaXtract-exported MS2 scan-header table containing the 'Total Ion Current' column per scan (CSV/TSV)

## Outputs

- qc_tic_comparison.csv summary table with per-run median MS1 and MS2 TIC values
- MS1 relative TIC decrease between underperforming and reference runs (percent)
- MS2 TIC reduction factor between underperforming and reference runs (fold change)

## How to apply

Run MetaXtract on the Thermo RAW files (e.g. via a YAML config with `scan_header.MS1` and `scan_header.MS2` enabled and the `Total Ion Current` column selected) to export per-scan scan-header tables. Load the MS1 and MS2 scan-header tables into pandas and compute the median 'Total Ion Current' per run (here four HeLa Pierce Protein Digest Standard QC runs: two underperforming 'UP' runs and two reference runs). For MS1, take the ratio of the median TIC of the UP runs to the reference runs and report the relative decrease (expected 8.4%); for MS2, derive the median-based reduction factor of UP versus reference runs (expected 9.2-fold). Aggregate both metrics with supporting per-run medians into a single summary table written to qc_tic_comparison.csv. A large MS1-vs-MS2 asymmetry (small MS1 drop, large MS2 reduction) is the diagnostic signal: it indicates an isolation/fragment transmission problem rather than overall ion generation failure, which is why both MS levels are compared rather than only total signal.

## Related tools

- **MetaXtract** (Extracts MS1 and MS2 scan-header tables (including 'Total Ion Current' per scan) directly from Thermo RAW files without database searching) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Computes per-run median TIC values and the UP-vs-reference ratio/fold-reduction metrics, then writes the summary CSV)

## Examples

```
python main.py --config /path/to/config.yml  # scan_header.MS1/MS2 with 'Total Ion Current': true, then: import pandas as pd; med = pd.read_csv('ms1_scan_headers.csv').groupby('run')['Total Ion Current'].median()
```

## Evaluation signals

- The MS1 median TIC ratio must reproduce the reported 8.4% relative decrease of UP runs versus reference runs (Figures 2a)
- The MS2 TIC reduction must reproduce the reported 9.2-fold factor (Figures 2b)
- qc_tic_comparison.csv exists and contains both headline metrics plus per-run median values for all four QC runs
- The TIC column in the exported scan headers is named 'Total Ion Current' (PSI-MS/mzML term MS:1000285) and is populated for every scan
- The comparison is median-based per run, not mean-based or per-scan, matching the workflow definition

## Limitations

- Only Thermo Fisher RAW files are supported; additional vendor formats are not yet supported
- Linking QC deviations to instrument-level causes (e.g. isolation/fragment transmission) is described as plausible rather than validated by the article
- MS-method and LC-method extraction options are not supported on Linux containers (use CLI/scan-header options instead); MetaXtract has no stated version number or changelog

## Evidence
