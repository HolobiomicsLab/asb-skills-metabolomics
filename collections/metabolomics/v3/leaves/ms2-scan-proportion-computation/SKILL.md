---
name: ms2-scan-proportion-computation
description: Use when QC-comparing the MS1/MS2 acquisition balance across Thermo RAW
  runs — e.g., diagnosing underperforming HeLa QC runs from MetaXtract scan-header
  CSVs where only the Scan Mode column is required.
when_to_use_negative:
- Identification-based QC is needed — this skill uses only acquisition-side scan headers
  and involves no database search results (unlike MSQC-style metrics).
- Input RAW files are not Thermo Fisher format — MetaXtract currently supports only
  Thermo RAW files.
- Only a single run is available with no comparison runs — the paper presents MS2
  proportion as a comparative indicator between reference and underperforming runs,
  not an absolute pass/fail threshold.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: MetaXtract
  role: Extracts MS1/MS2 scan headers with the Scan Mode column from Thermo RAW files
    into per-run structured CSV/TSV exports consumed by this computation
  repo: https://github.com/Rappsilber-Laboratory/MetaXtract
- name: pandas
  role: Loads each run's scan-header export, classifies rows by Scan Mode, computes
    MS2 count / total scans × 100, and writes ms2_scan_proportions.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.11.12.687968
    title: 'MetaXtract: Extracting Metadata from Raw Files for FAIR Data Practices
      and Workflow Optimisation'
  source_content_hash: sha256:71f0aeb13cfacdb8677e3016a10c3aee6aa851d05471df683780cbc3599adb7a
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ms2-scan-proportion-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the percentage of MS2 scans relative to total scans per Thermo RAW run from MetaXtract scan-header exports (Scan Mode column) using pandas, providing a search-free acquisition-quality indicator that separated the paper's reference HeLa QC runs (60%, 62%) from underperforming runs (56%, 59%).

## When to use

Use when QC-comparing the MS1/MS2 acquisition balance across Thermo RAW runs — e.g., diagnosing underperforming HeLa QC runs from MetaXtract scan-header CSVs where only the Scan Mode column is required.

## When NOT to use

- Identification-based QC is needed — this skill uses only acquisition-side scan headers and involves no database search results (unlike MSQC-style metrics).
- Input RAW files are not Thermo Fisher format — MetaXtract currently supports only Thermo RAW files.
- Only a single run is available with no comparison runs — the paper presents MS2 proportion as a comparative indicator between reference and underperforming runs, not an absolute pass/fail threshold.

## Inputs

- MetaXtract MS1/MS2 scan-header CSV/TSV exports with the 'Scan Mode' column selected, one per Thermo RAW run
- A set of Thermo RAW-file runs to compare (reference vs underperforming, e.g., Ref1/Ref2/UP1/UP2 HeLa QC runs acquired on Pierce HeLa Protein Digest Standard)

## Outputs

- ms2_scan_proportions.csv — one row per run listing MS2 scan count, total scan count, and MS2 proportion in percent (e.g., 60, 62, 56, 59)

## How to apply

Run MetaXtract on each Thermo RAW file with scan-header extraction enabled and the `Scan Mode` column selected for both MS1 and MS2 (or `select_all: true` in config.yml), producing structured per-run CSV/TSV exports. Load each export with pandas and classify every scan row as MS1 or MS2 from its Scan Mode value so that MS1 and MS2 counts partition the run's total scans. Compute the MS2 fraction per run as MS2 count / total scans × 100 and compare across runs: in the paper's Pierce HeLa Protein Digest Standard demonstration (DDA top-ten, HCD at normalized collision energy 27%), reference runs measured 60% and 62% MS2 versus 56% and 59% for underperforming runs despite a similar total number of MS1 and MS2 scans, so the proportion — not the absolute count — carries the QC signal. Write one summary row per run (MS2 scan count, total scan count, MS2 proportion) to ms2_scan_proportions.csv, then verify the recomputed percentages reproduce the reported Figure S3 values exactly.

## Related tools

- **MetaXtract** (Extracts MS1/MS2 scan headers with the Scan Mode column from Thermo RAW files into per-run structured CSV/TSV exports consumed by this computation) — https://github.com/Rappsilber-Laboratory/MetaXtract
- **pandas** (Loads each run's scan-header export, classifies rows by Scan Mode, computes MS2 count / total scans × 100, and writes ms2_scan_proportions.csv)

## Examples

```
frac = df['Scan Mode'].str.contains('ms2', case=False).sum() / len(df) * 100  # per run; write row to ms2_scan_proportions.csv
```

## Evaluation signals

- Recomputed MS2 proportions exactly reproduce the reported Figure S3 values: 60% (Ref1), 62% (Ref2), 56% (UP1), 59% (UP2).
- MS1 row count + MS2 row count equals the run's total scan count; no scan rows remain unclassified after Scan Mode parsing.
- Underperforming runs rank below reference runs on MS2 proportion even though total MS1 and MS2 scan counts are similar across all four runs.
- ms2_scan_proportions.csv contains exactly one row per run with columns for MS2 scan count, total scan count, and MS2 proportion.
- Scan-header exports were generated with Scan Mode selected for both MS1 and MS2 (PSI-MS-named column present in each export).

## Limitations

- MetaXtract currently supports only Thermo Fisher RAW files; additional vendor file formats are not yet supported.
- The article describes linking QC deviations to instrument-level causes as plausible rather than validated, so a low MS2 proportion identifies an underperforming run only comparatively.
- Reported proportions are at integer granularity (56–62%); the margin between runs is a few percentage points, so total scan counts should be reported alongside.

## Evidence

- [supplementary] Figure S3 finding reproduced by this skill: [evidence span withheld — anchor status: not_found]
