---
name: lcms-total-ion-current-qc-computation
description: 'Use when MetaXtract scan-header tables from Thermo RAW files contain
  a ''Total Ion Current'' column for MS1/MS2 and you need search-free run-level QC:
  comparing UP vs reference HeLa QC runs or gating runs by MS2/MS1 TIC ratio.'
when_to_use_negative:
- The QC question requires identification-based metrics — e.g. MSQC-style metrics
  that need database search results — rather than acquisition-side TIC signals.
- Input files are not Thermo Fisher RAW (MetaXtract currently supports only Thermo
  RAW; other vendor formats are not yet supported).
- You need per-peak or spectral content (m/z/intensity arrays); this skill operates
  only on scan-header-level Total Ion Current columns.
tools:
- name: MetaXtract
  role: Extracts per-scan Total Ion Current from Thermo RAW scan headers for MS1 and
    MS2 via RawFileReader; run in CLI mode with a YAML config enabling the 'Total
    Ion Current' column under scan_header.MS1 and scan_header.MS2
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Loads the exported MS1/MS2 scan-header tables keyed by scan number and Scan
    Start Time (min), computes per-run median/sum TIC, the MS2/MS1 ratio, and writes
    qc_tic_comparison.csv and ms2_tic_flag_report.csv
- name: PyYAML
  role: Parses the YAML configuration supplying the user-configurable ms2_tic_min_percent
    flagging threshold
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:7a9178de5b1656cf73efcc1843c67a4925231b4a51e7adb46bde50b7583b5c0c
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# lcms-total-ion-current-qc-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identification-free LC-MS run-level QC based on Total Ion Current (TIC) statistics extracted from Thermo RAW scan headers. It quantifies MS1 and MS2 TIC differences between reference and underperforming runs (e.g. the 8.4% MS1 decrease and 9.2-fold MS2 reduction in HeLa Pierce digest QC runs) and can flag runs whose MS2 TIC falls below a configurable percentage of MS1 TIC before database searching.

## When to use

Use when MetaXtract scan-header tables from Thermo RAW files contain a 'Total Ion Current' column for MS1/MS2 and you need search-free run-level QC: comparing UP vs reference HeLa QC runs or gating runs by MS2/MS1 TIC ratio.

## When NOT to use

- The QC question requires identification-based metrics — e.g. MSQC-style metrics that need database search results — rather than acquisition-side TIC signals.
- Input files are not Thermo Fisher RAW (MetaXtract currently supports only Thermo RAW; other vendor formats are not yet supported).
- You need per-peak or spectral content (m/z/intensity arrays); this skill operates only on scan-header-level Total Ion Current columns.

## Inputs

- Thermo Fisher .RAW files (e.g. four HeLa Pierce Protein Digest Standard QC runs)
- MetaXtract-exported MS1 scan-header tables containing the 'Total Ion Current' column (PSI-MS MS:1000285)
- MetaXtract-exported MS2 scan-header tables containing the 'Total Ion Current' column
- YAML configuration file specifying scan_header.MS1/MS2 columns and, for flagging, an ms2_tic_min_percent threshold

## Outputs

- qc_tic_comparison.csv: summary table with per-run median MS1 and MS2 TIC, the MS1 relative decrease (8.4%), and the MS2 reduction factor (9.2-fold)
- ms2_tic_flag_report.csv: one row per RAW file with MS2/MS1 TIC ratio, configured threshold, and pass/fail flag status
- Flag/terminate decision used to gate runs before database searching

## How to apply

Enable scan-header extraction for both MS levels in the MetaXtract YAML config by setting Total Ion Current: true under scan_header.MS1 and scan_header.MS2, then load the exported scan-header tables with pandas keyed by scan number and Scan Start Time (min). For each run, aggregate per-scan TIC (median over scans) separately for MS1 and MS2; for the diagnosis workflow, take the ratio of median MS1 TIC of the two underperforming (UP) runs to the two reference runs and report the relative decrease (8.4%), and analogously derive the MS2 reduction factor (9.2-fold). For the automated-flagging extension, compute a single MS2/MS1 TIC ratio per processed RAW file and compare it against a user-configurable percentage threshold (e.g. ms2_tic_min_percent in the YAML), writing ms2_tic_flag_report.csv with ratio, threshold, and pass/fail status to gate the run before downstream database searching. Aggregate both metrics plus per-run medians into qc_tic_comparison.csv for the comparison workflow. Rationale: TIC is an acquisition-side signal that requires no peptide identification, so the decision can be made in near real time and terminates wasted searching.

## Related tools

- **MetaXtract** (Extracts per-scan Total Ion Current from Thermo RAW scan headers for MS1 and MS2 via RawFileReader; run in CLI mode with a YAML config enabling the 'Total Ion Current' column under scan_header.MS1 and scan_header.MS2) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Loads the exported MS1/MS2 scan-header tables keyed by scan number and Scan Start Time (min), computes per-run median/sum TIC, the MS2/MS1 ratio, and writes qc_tic_comparison.csv and ms2_tic_flag_report.csv)
- **PyYAML** (Parses the YAML configuration supplying the user-configurable ms2_tic_min_percent flagging threshold)

## Examples

```
python main.py --config /path/to/config.yml
```

## Evaluation signals

- The MS1 median TIC ratio of UP vs reference runs reproduces the reported 8.4% relative decrease (Figure 2a).
- The MS2 median TIC comparison reproduces the reported 9.2-fold reduction in UP runs (Figure 2b).
- ms2_tic_flag_report.csv exists as valid CSV and contains ratio and flag columns; every ratio falls within [0, 1].
- Every flagged status is exactly (ratio < configured ms2_tic_min_percent threshold) — no discrepancy between ratio and flag.
- qc_tic_comparison.csv includes supporting per-run median TIC values for all four HeLa QC runs, not only the summary ratios.

## Limitations

- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported.
- Linking QC deviations to instrument-level causes is described as plausible rather than validated — TIC drops indicate an isolation/fragment transmission problem but the causal chain is inferential.
- No version number is stated for MetaXtract, and no changelog is provided, which complicates reproducibility of extraction behaviour across releases.

## Evidence
