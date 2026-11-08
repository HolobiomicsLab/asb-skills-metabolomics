---
name: cross-ms-level-tic-ratio-analysis
description: Use when Thermo RAW files have been processed with MetaXtract and you
  must automatically flag acquisitions whose MS2 TIC drops below a set fraction of
  MS1 TIC, without running identifications.
when_to_use_negative:
- Input is not Thermo Fisher RAW data — MetaXtract currently supports only Thermo
  RAW files; additional vendor formats are not yet supported
- Database-search-derived QC metrics are required, e.g. identification-based metrics
  like those MSQC computes from search results — this skill is identification-free
  and uses only acquisition-side TIC signals
- You need fine-grained, scan-level anomaly localization rather than a single per-run
  ratio gate — this skill aggregates TIC over the whole run
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaXtract
  role: Extracts MS1 and MS2 scan-header metadata, including Total Ion Current and
    Scan Start Time (min), from Thermo RAW files via CLI with a YAML config
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Loads the exported MS1/MS2 scan-header tables, aggregates per-run TIC, computes
    MS2/MS1 ratios, and writes the flag report CSV
- name: PyYAML
  role: Parses the YAML config supplying the ms2_tic_min_percent threshold and MetaXtract
    scan-header settings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:d2662278f2952f1e1ff3f8665fab02d5b22e5557c2d09bcc55841fafd331fa73
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-ms-level-tic-ratio-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-run MS2/MS1 total ion current (TIC) ratios from MetaXtract scan-header exports and flag underperforming acquisitions against a configurable threshold, enabling identification-free, near real-time QC gating before database searching.

## When to use

Use when Thermo RAW files have been processed with MetaXtract and you must automatically flag acquisitions whose MS2 TIC drops below a set fraction of MS1 TIC, without running identifications.

## When NOT to use

- Input is not Thermo Fisher RAW data — MetaXtract currently supports only Thermo RAW files; additional vendor formats are not yet supported
- Database-search-derived QC metrics are required, e.g. identification-based metrics like those MSQC computes from search results — this skill is identification-free and uses only acquisition-side TIC signals
- You need fine-grained, scan-level anomaly localization rather than a single per-run ratio gate — this skill aggregates TIC over the whole run

## Inputs

- Thermo Fisher RAW files processed by MetaXtract CLI with a YAML config
- MetaXtract-exported MS1 scan-header table (Total Ion Current, Scan Start Time (min) columns)
- MetaXtract-exported MS2 scan-header table (Total Ion Current, Scan Start Time (min) columns)
- YAML config supplying the MS2 TIC threshold as a percentage of MS1 TIC (e.g. ms2_tic_min_percent)

## Outputs

- ms2_tic_flag_report.csv with one row per processed RAW file containing the MS2/MS1 TIC ratio, the configured threshold, and a pass/fail flag status
- Gate decision used to terminate underperforming runs before database searching

## How to apply

Run MetaXtract in CLI mode with a YAML config that enables scan-header extraction for both MS levels with the 'Total Ion Current' column selected (Total Ion Current: true under scan_header.MS1 and scan_header.MS2), plus 'Scan Start Time (min)' for keying. Load the exported MS1 and MS2 scan-header tables with pandas, keyed by scan number and Scan Start Time (min). Aggregate per-run MS1 TIC and MS2 TIC (sum or median over scans) and compute the MS2/MS1 TIC ratio for each processed RAW file. Compare each run's ratio against the user-configurable threshold from the YAML config (e.g. ms2_tic_min_percent) and write ms2_tic_flag_report.csv with one row per RAW file containing ratio, threshold, and pass/fail status to gate the run before downstream searching. The rationale is that MetaXtract's acquisition-side signals allow search-free, near real-time performance monitoring.

## Related tools

- **MetaXtract** (Extracts MS1 and MS2 scan-header metadata, including Total Ion Current and Scan Start Time (min), from Thermo RAW files via CLI with a YAML config) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Loads the exported MS1/MS2 scan-header tables, aggregates per-run TIC, computes MS2/MS1 ratios, and writes the flag report CSV)
- **PyYAML** (Parses the YAML config supplying the ms2_tic_min_percent threshold and MetaXtract scan-header settings)

## Examples

```
python main.py --config /path/to/config.yml  # then with pandas: ratio = ms2_tic.sum() / ms1_tic.sum(); flag = ratio < threshold
```

## Evaluation signals

- ms2_tic_flag_report.csv exists and parses as valid CSV with ratio and flag columns
- All ratio values fall within [0, 1]
- Every flagged status is exactly (ratio < configured threshold) — i.e. flag status is a deterministic function of ratio vs threshold
- Report contains one row per processed RAW file, matching MetaXtract's processed input count
- Exported scan-header tables include the Total Ion Current and Scan Start Time (min) columns as configured in the YAML

## Limitations

- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported
- No version number stated for MetaXtract; repository has no changelog
- Aggregation choice (sum vs median over scans) and the ratio threshold are user-configurable; the paper does not prescribe a universal cutoff
- MS-method and LC-method export is not supported on Linux containers, so related method metadata may be unavailable in containerized pipelines

## Evidence

- [other] scan-header extraction for both MS levels with the Total Ion Current column selected: [evidence span withheld — anchor status: not_found]
- [other] flag/terminate runs by MS2/MS1 TIC threshold: [evidence span withheld — anchor status: not_found]
- [abstract] search-free near real-time monitoring from acquisition-side signals: "MetaXtract enables search-free, near real-time performance monitoring by relying on acquisition-side signals, providing actionable indicators immediately after data acquisition"
- [readme] README Total Ion Current column availability for MS1 and MS2 scan headers: "Total Ion Current: true
      Scan Start Time (min): true
      Scan Mode: false
  MS2:
    select_all: true"
- [readme] README column naming for Total Ion Current: "| `Total Ion Current` | `total ion current`, `MS:1000285` | Direct match. |"
