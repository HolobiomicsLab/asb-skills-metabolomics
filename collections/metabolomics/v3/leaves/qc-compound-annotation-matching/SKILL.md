---
name: qc-compound-annotation-matching
description: Use when LC-IM-MS derived CCS values for known benchmark/QC compounds
  (e.g., Quad LCMS QC mixture) need to be validated against external reference databases
  such as the CCS Compendium.
when_to_use_negative:
- The compounds of interest are not part of an external reference database (e.g.,
  no CCS Compendium entries exist), so no bias benchmarking is possible
- CCS values come from a different IM-MS technique (e.g., TWIM-MS rather than drift
  tube IM-MS with nitrogen drift gas), making direct DTCCS N2 comparison against Compendium
  values inappropriate
- Input is raw mzML profile data requiring CCSfind processing itself rather than already-determined
  CCS values ready for benchmarking
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CCSfind
  role: Source of the experimentally determined DTCCS N2 values for the Quad LCMS
    QC compounds being benchmarked
  repo: https://github.com/sangeeta97/ccs_find.git
- name: pandas
  role: Loads Table 1 DTCCS N2 values and Compendium records, matches compounds, and
    writes ccs_bias_results.csv
- name: numpy
  role: Computes absolute percent CCS bias per compound and the mean absolute bias
    across the eight compounds
- name: XlsxWriter
  role: Backend for writing the per-compound and mean absolute bias results table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/jms.5040
    title: 'CCSfind: A tool for chemically informed LC‐IM‐MS database building'
  source_content_hash: sha256:98b8c94fc94699cb11f014786741073a391281db4b533272c995737b382eb56d
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
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# qc-compound-annotation-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks experimentally determined drift-tube CCS values (DTCCS N2) from CCSfind against published CCS Compendium reference entries for the same small-molecule QC compounds, computing per-compound and mean absolute percent bias to validate standardized DTIM-MS calibration.

## When to use

Use when LC-IM-MS derived CCS values for known benchmark/QC compounds (e.g., Quad LCMS QC mixture) need to be validated against external reference databases such as the CCS Compendium.

## When NOT to use

- The compounds of interest are not part of an external reference database (e.g., no CCS Compendium entries exist), so no bias benchmarking is possible
- CCS values come from a different IM-MS technique (e.g., TWIM-MS rather than drift tube IM-MS with nitrogen drift gas), making direct DTCCS N2 comparison against Compendium values inappropriate
- Input is raw mzML profile data requiring CCSfind processing itself rather than already-determined CCS values ready for benchmarking

## Inputs

- Table 1 DTCCS N2 values for the eight Quad LCMS QC compounds (from CCSfind output on LC-DTIM-MS mzML data)
- CCS Compendium reference entries (CCS values) for the same benchmark small-molecule compounds
- CCSfind deposit from the BOKU drive link and/or the sangeeta97/ccs_find GitHub repository with example data

## Outputs

- Per-compound absolute percent CCS bias values (Quad LCMS QC compound vs. CCS Compendium reference)
- Mean absolute bias across the eight compounds, compared against the reported ~1.1% to 0.51% range
- ccs_bias_results.csv written via pandas/XlsxWriter

## How to apply

Obtain the CCSfind deposit from the BOKU drive link or clone the sangeeta97/ccs_find GitHub repository to access the tool and its example data. Load the Table 1 DTCCS N2 values for the eight Quad LCMS QC compounds with pandas, then retrieve and extract the corresponding reference CCS values from the CCS Compendium for those same compounds. Match each Compendium record to its Quad LCMS QC compound and compute the absolute percent CCS bias against the CCSfind DTCCS N2 value using numpy. Average the absolute biases across the eight compounds and compare against the reported range of about 1.1% down to 0.51%, which is the expectation for DTIM-MS measurements using a standardized single-field CCS calibration (Tfix and Beta coefficients, nitrogen drift gas). Write per-compound and mean absolute bias values to ccs_bias_results.csv via pandas/XlsxWriter.

## Related tools

- **CCSfind** (Source of the experimentally determined DTCCS N2 values for the Quad LCMS QC compounds being benchmarked) — https://github.com/sangeeta97/ccs_find.git
- **pandas** (Loads Table 1 DTCCS N2 values and Compendium records, matches compounds, and writes ccs_bias_results.csv)
- **numpy** (Computes absolute percent CCS bias per compound and the mean absolute bias across the eight compounds)
- **XlsxWriter** (Backend for writing the per-compound and mean absolute bias results table)

## Examples

```
python processor.py -g config.ini  # then in Python: df = pd.read_csv('ccs_bias_results.csv'); mean_bias = np.mean(np.abs(df['bias_pct']))
```

## Evaluation signals

- Mean absolute bias across all eight Quad LCMS QC compounds falls within the reported range of approximately 1.1% down to 0.51%
- Every CCS Compendium record is matched to exactly one Quad LCMS QC compound before bias computation (no unmatched or duplicated annotations)
- ccs_bias_results.csv contains both per-compound and mean absolute bias columns and opens cleanly via pandas/XlsxWriter
- CCS values being compared were determined with single-field CCS calibration coefficients (Tfix and Beta) and nitrogen as the drift gas, matching the CCSfind DTCCS N2 convention
- Bias values are computed as absolute percent differences, so all reported biases are non-negative

## Limitations

- CCS Compendium coverage is incomplete — only compounds with available Compendium data can be benchmarked, so not all Table 1 entries may have a reference match
- Agreement is only as good as the standardized calibration approach; the ~1.1% to 0.51% bias expectation is specific to DTIM-MS with single-field CCS calibration (Tfix, Beta) in nitrogen
- CCSfind requires specific inputs including profile mzML files from LC-DTIM-MS, drift gas identity, and primary ion species; missing any of these prevents generating the CCS values to be benchmarked

## Evidence

- [other] Compendium bias benchmarking workflow: [evidence span withheld — anchor status: not_found]
