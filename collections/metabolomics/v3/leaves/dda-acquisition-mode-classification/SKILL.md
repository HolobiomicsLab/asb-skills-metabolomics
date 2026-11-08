---
name: dda-acquisition-mode-classification
description: Use when comparing Thermo RAW-file runs by scan-level acquisition composition
  (MS2/total scans) using MetaXtract scan-header exports, e.g. diagnosing underperforming
  DDA HeLa QC runs against reference runs via Figure S3-style proportions.
when_to_use_negative:
- RAW files from vendors other than Thermo Fisher — MetaXtract currently supports
  only Thermo RAW files via RawFileReader
- QC questions that require database search results (e.g. peptide ID rates) — this
  skill relies solely on acquisition-side scan metadata
- Scan-header exports where the 'Scan Mode' column was not selected, so scan type
  cannot be determined per row
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: MetaXtract
  role: Extracts MS1/MS2 scan headers (including Scan Mode) from Thermo RAW files
    into CSV/TSV used for scan classification and MS2-fraction computation
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Loads the MetaXtract scan-header exports and computes per-run MS2 counts,
    total counts, and proportions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:49f29e2a3d5f5cac8adc2035fbbb0260661b70ab9fefeb7b24e87fd8602f39b4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# dda-acquisition-mode-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify scans from MetaXtract RAW-file exports as MS1 or MS2 using the Scan Mode column, then compute the per-run MS2 fraction (MS2 count / total scans × 100) to benchmark acquisition behaviour across runs, e.g. reference vs underperforming HeLa QC runs.

## When to use

Use when comparing Thermo RAW-file runs by scan-level acquisition composition (MS2/total scans) using MetaXtract scan-header exports, e.g. diagnosing underperforming DDA HeLa QC runs against reference runs via Figure S3-style proportions.

## When NOT to use

- RAW files from vendors other than Thermo Fisher — MetaXtract currently supports only Thermo RAW files via RawFileReader
- QC questions that require database search results (e.g. peptide ID rates) — this skill relies solely on acquisition-side scan metadata
- Scan-header exports where the 'Scan Mode' column was not selected, so scan type cannot be determined per row

## Inputs

- Thermo Fisher RAW files for the runs to compare (e.g. Ref1, Ref2, UP1, UP2 HeLa QC runs)
- MetaXtract scan-header CSV/TSV exports with the 'Scan Mode' column selected for MS1 and MS2
- config.yml with file_based_details and complete-ms1/complete-ms2 outputs enabled

## Outputs

- ms2_scan_proportions.csv with one row per run listing MS2 scan count, total scan count, and MS2 proportion (%)
- Per-run classification of every scan as MS1 or MS2 based on Scan Mode

## How to apply

Run MetaXtract on the four RAW files (Ref1, Ref2, UP1, UP2) with the Scan Mode column enabled for MS1 and MS2 scan headers in the config (set scan_header.MS1.columns.Scan Mode: true and scan_header.MS2.columns.Scan Mode: true). Load the scan-header exports with pandas and classify each scan row as MS1 or MS2 from the Scan Mode value. Count MS2 scans and total scans per run and compute the MS2 proportion (MS2 count / total scans × 100). Reproduce the reported values of 60% (Ref1), 62% (Ref2), 56% (UP1), and 59% (UP2) from Figure S3; note the four runs have a similar total number of MS1 and MS2 scans, so the diagnostic signal is the proportion, not raw counts. Because the runs use top-ten DDA (the ten most intense 2+–6+ ions fragmented by HCD at NCE 27%), deviations in MS2 fraction indicate acquisition-side duty-cycle issues rather than identification failure, keeping the check search-free.

## Related tools

- **MetaXtract** (Extracts MS1/MS2 scan headers (including Scan Mode) from Thermo RAW files into CSV/TSV used for scan classification and MS2-fraction computation) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Loads the MetaXtract scan-header exports and computes per-run MS2 counts, total counts, and proportions)

## Examples

```
import pandas as pd; df = pd.read_csv('Ref1_ms2_scan_headers.csv', usecols=['Scan Mode']); frac = (df['Scan Mode'].astype(str).str.contains('MS2')).mean()*100
```

## Evaluation signals

- Computed MS2 proportions match the Figure S3 values: 60% (Ref1), 62% (Ref2), 56% (UP1), 59% (UP2)
- MS1 count + MS2 count equals the total scan count for every run
- Summary table has exactly one row per run with columns MS2 scan count, total scan count, and MS2 proportion
- Underperforming runs (UP1, UP2) show strictly lower MS2 fractions than both reference runs, consistent with the expected ordering
- 'Scan Mode' column is present and non-null in every loaded scan-header file

## Limitations

- MetaXtract currently supports only Thermo Fisher RAW files; other vendor formats are not supported
- Reported proportions are rounded integers (56–62%), so exact scan counts cannot be recovered from the figure values alone — only the proportion ordering can be verified
- Linking QC deviations such as reduced MS2 fraction to specific instrument-level causes is described as plausible rather than validated
- MS-method and LC-method exports are not supported on Linux containers, so containerised runs should rely on scan-header outputs

## Evidence

- [discussion] Vendor support limitation: [evidence span withheld — anchor status: not_found]
