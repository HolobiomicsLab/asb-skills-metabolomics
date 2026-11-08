---
name: lipidomics-ccs-correction-application
description: Use when raw negative-mode TIMS CCS measurements from constituent datasets
  (e.g. Vasilopoulou et al., Tsugawa et al.) must be linearly calibrated toward DT
  reference CCS values and the post-correction MDRE reported.
when_to_use_negative:
- Input CCS values are already calibrated to DT references — re-applying linear corrections
  would double-correct and bias the MDRE
- Data are HILIC retention times or m/z rather than CCS measurements — this skill's
  correction equations and MDRE metric apply only to collision cross sections
tools:
- name: LiPydomics
  role: Python package providing the CCS prediction and lipidomics analysis framework
    within which the corrected CCS tables and error statistics are computed
  repo: github.com/dylanhross/lipydomics
- name: pandas
  role: Loads the constituent TIMS CCS datasets into DataFrames and concatenates corrected
    CCS rows into the single corrected table written to CSV
- name: Python
  role: Scripting environment in which the linear correction equations are applied
    and the MDRE is computed
- name: SciPy
  role: Numerical routines supporting the linear correction fitting and error computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:d092e0150a35b463d7d3cd0efe8c89992629f3861f74548c22dadc1b6cfc17f3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - SciPy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lipidomics-ccs-correction-application

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies instrument-specific linear CCS correction equations (e.g. Figure S2 calibrations) to raw trapped ion mobility spectrometry (TIMS) collision cross section values, calibrating them toward drift-tube (DT) reference values and quantifying calibration quality via median relative error (MDRE).

## When to use

Use when raw negative-mode TIMS CCS measurements from constituent datasets (e.g. Vasilopoulou et al., Tsugawa et al.) must be linearly calibrated toward DT reference CCS values and the post-correction MDRE reported.

## When NOT to use

- Input CCS values are already calibrated to DT references — re-applying linear corrections would double-correct and bias the MDRE
- Data are HILIC retention times or m/z rather than CCS measurements — this skill's correction equations and MDRE metric apply only to collision cross sections

## Inputs

- Negative-mode TIMS CCS measurement tables from constituent datasets (Vasilopoulou et al., Tsugawa et al.)
- Dataset-specific linear correction equations (Figure S2 calibrations)
- DT reference CCS values paired to each TIMS measurement

## Outputs

- corrected_ccs_table.csv — one corrected CCS entry per input measurement
- Computed post-correction MDRE value, recorded alongside the reported −0.36% for comparison

## How to apply

Load the constituent TIMS CCS measurement tables (e.g. the negative-mode CCS values from Vasilopoulou et al. and Tsugawa et al.) into pandas DataFrames. For each dataset, select the dataset-specific linear correction equation — for the LiPydomics companion calibration this is the equation reported in Figure S2 — and apply it to the raw CCS values to calibrate them toward DT reference values. Concatenate the corrected CCS rows from all datasets into a single table so every input measurement retains a corrected entry. Compute the post-correction median relative error (MDRE) of corrected CCS values against their DT references; for the negative-mode TIMS data the reported MDRE is −0.36%, so a computed value within reasonable numerical tolerance of −0.36% validates the applied corrections.

## Related tools

- **LiPydomics** (Python package providing the CCS prediction and lipidomics analysis framework within which the corrected CCS tables and error statistics are computed) — github.com/dylanhross/lipydomics
- **pandas** (Loads the constituent TIMS CCS datasets into DataFrames and concatenates corrected CCS rows into the single corrected table written to CSV)
- **Python** (Scripting environment in which the linear correction equations are applied and the MDRE is computed)
- **SciPy** (Numerical routines supporting the linear correction fitting and error computation)

## Examples

```
import pandas as pd; df = pd.concat([vas, tsu]); mdre = ((df.ccs_corrected - df.ccs_dt) / df.ccs_dt).median(); df.to_csv('corrected_ccs_table.csv')
```

## Evaluation signals

- Computed negative-mode TIMS MDRE reproduces the reported post-correction value of −0.36% within reasonable numerical tolerance
- corrected_ccs_table.csv contains a corrected CCS entry for every input measurement (row-count invariant)
- Every corrected CCS value derives from exactly one dataset-specific linear equation matched to its source dataset
- MDRE is computed as the median of signed relative errors against paired DT reference values, not mean or absolute median

## Limitations

- The correction equations are dataset-specific (Figure S2 calibrations); applying one dataset's equation to another dataset's measurements is invalid
- Validation tolerance depends on the precision of the reported −0.36% benchmark; discrepancies may reflect rounding in the published figure rather than implementation error
- The MDRE is a median signed error — it masks outlier corrections that could be individually large

## Evidence
