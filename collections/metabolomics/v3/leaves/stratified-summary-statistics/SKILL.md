---
name: stratified-summary-statistics
description: Use when you have paired CCS (or other instrument-reported quantity)
  measurements from two methods/software for the same compounds and need per-stratum
  (mode, acquisition setting) average absolute relative bias summaries and a tabulated
  output.
when_to_use_negative:
- Input is raw LC-DTIM-MS profile data in mzML that still needs CCS extraction — use
  the CCSfind pipeline (peak picking with isotopologue matching confirmation, single-field
  CCS calibration with Tfix and Beta coefficients) first, not bias statistics
- The two CCS value sets are not paired per compound (different compounds or no shared
  identifiers), making per-entry relative bias meaningless
- A single pooled average is requested with no stratification by ionization mode or
  acquisition setting — mode-specific structure (e.g., 0.35%–0.93% single-pulse bias)
  would be masked
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the vendor-vs-CCSfind comparison table, computes per-compound absolute
    relative bias columns, and computes groupby stratum means for the summary table
- name: numpy
  role: Vectorized elementwise arithmetic for the |CCSfind − vendor| / vendor × 100%
    bias computation across all Table 1 entries
- name: XlsxWriter
  role: Output engine used with pandas to write the stratified ccs_bias_summary.csv/Excel
    summary table
- name: CCSfind
  role: Source of the targeted DTCCS N2 values being compared against vendor non-targeted
    values in the stratified bias computation
  repo: https://github.com/sangeeta97/ccs_find.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:27ca1b696b4c02becd658dc08b7419272a7d944a38bb3439290d62bf83d145a2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sangeeta97/ccs_find
  repo_url: https://github.com/sangeeta97/ccs_find.git
  tools:
  - CCSfind
  - Python
  - XlsxWriter
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# stratified-summary-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A pandas/numpy workflow for computing stratified average absolute relative bias between paired CCS values — here CCSfind (targeted) vs vendor (non-targeted) DTCCS N2 values — across ionization modes and acquisition settings, reproducing Table 1 summary statistics from the CCSfind paper.

## When to use

Use when you have paired CCS (or other instrument-reported quantity) measurements from two methods/software for the same compounds and need per-stratum (mode, acquisition setting) average absolute relative bias summaries and a tabulated output.

## When NOT to use

- Input is raw LC-DTIM-MS profile data in mzML that still needs CCS extraction — use the CCSfind pipeline (peak picking with isotopologue matching confirmation, single-field CCS calibration with Tfix and Beta coefficients) first, not bias statistics
- The two CCS value sets are not paired per compound (different compounds or no shared identifiers), making per-entry relative bias meaningless
- A single pooled average is requested with no stratification by ionization mode or acquisition setting — mode-specific structure (e.g., 0.35%–0.93% single-pulse bias) would be masked

## Inputs

- Table 1-style comparison table of vendor (non-targeted) and CCSfind (targeted) DTCCS N2 values per compound, with ionization mode and acquisition-setting labels (CSV/Excel readable by pandas via xlrd)
- Stratification metadata per entry: ionization mode (positive/negative) and acquisition setting (e.g., 4IMS/HR, single-pulse)

## Outputs

- Per-compound absolute relative bias values (%) in the vendor-vs-CCSfind DTCCS N2 comparison
- Stratified summary table of average absolute bias per stratum (all data, positive 4IMS/HR, negative mode, single-pulse), written to ccs_bias_summary.csv

## How to apply

Load the vendor-vs-CCSfind DTCCS N2 comparison table (Table 1) into a pandas DataFrame with columns identifying compound, ionization mode, and acquisition setting. For each compound entry compute the absolute relative bias as |CCSfind − vendor| / vendor × 100%. Stratify the per-compound biases by acquisition condition: all data, positive mode 4IMS/HR, negative mode, and single-pulse standard settings, then average absolute biases within each stratum using numpy/pandas groupby-mean. Verify the strata against the reported values — ~0.15% average over all data, 0.08% average for positive-mode 4IMS/HR, 0.16% average for negative mode, and larger single-pulse biases of 0.35%–0.93% — as a sanity check that pairing and stratification are correct. Write the stratified summary table to ccs_bias_summary.csv using pandas with the XlsxWriter engine.

## Related tools

- **pandas** (Loads the vendor-vs-CCSfind comparison table, computes per-compound absolute relative bias columns, and computes groupby stratum means for the summary table)
- **numpy** (Vectorized elementwise arithmetic for the |CCSfind − vendor| / vendor × 100% bias computation across all Table 1 entries)
- **XlsxWriter** (Output engine used with pandas to write the stratified ccs_bias_summary.csv/Excel summary table)
- **CCSfind** (Source of the targeted DTCCS N2 values being compared against vendor non-targeted values in the stratified bias computation) — https://github.com/sangeeta97/ccs_find.git

## Examples

```
df['bias_pct']=(df.ccsfind_dtccs_n2-df.vendor_dtccs_n2).abs()/df.vendor_dtccs_n2*100; df.groupby(['mode','acq'])['bias_pct'].mean().to_csv('ccs_bias_summary.csv')
```

## Evaluation signals

- All-data average absolute bias computes to ~0.15%
- Positive-mode 4IMS/HR stratum averages ~0.08% and negative-mode stratum averages ~0.16%
- Single-pulse standard-settings stratum biases fall in the 0.35%–0.93% range
- Per-compound bias values are non-negative and defined only for entries where the vendor DTCCS N2 denominator is non-zero
- ccs_bias_summary.csv contains one row per stratum with a numeric average absolute bias column

## Limitations

- The reported biases compare CCSfind targeted processing against vendor non-targeted processing, so the stratum averages reflect that acquisition asymmetry rather than pure software differences
- Single-pulse standard settings show substantially larger biases (0.35%–0.93%) than multi-IMS/HR acquisitions, so pooling across acquisition settings hides this structure
- Requires paired vendor and CCSfind values for the same compounds under the same instrument conditions (drift gas, e.g., nitrogen, and single-field CCS calibration coefficients Tfix and Beta must match)

## Evidence
