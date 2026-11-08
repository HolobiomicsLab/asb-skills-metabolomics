---
name: median-relative-error-computation
description: Use when a collection of corrected/predicted CCS measurements with reference
  values needs its central accuracy quantified as a median relative error (MDRE),
  or when validating a calibration by reproducing a reported MDRE.
when_to_use_negative:
- The CCS values are already calibrated toward DT reference values and no linear correction
  from Figure S2 needs to be applied — a plain relative-error summary suffices without
  the calibration step
- The measurements are positive-mode or drift-gas conditions other than TIMS with
  the negative-mode linear corrections; the Figure S2 equations would not be applicable
- The goal is CCS prediction itself (e.g., training ML prediction models for the 22/23
  lipid classes) rather than validating corrected measurements against DT references
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3954
tools:
- name: LiPydomics
  role: Python package providing the lipidomics analysis environment within which
    the CCS correction and MDRE computation are performed
  repo: https://github.com/dylanhross/lipydomics
- name: pandas
  role: Loads the constituent CCS datasets into DataFrames, concatenates corrected
    values into a single table, and computes the median relative error
- name: SciPy
  role: Supports the numerical/scientific computing routines underlying the correction
    and error calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:3b1984495876c9fcdaf5e30f7fd240443b57cbdee23155081895483b5f347004
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dylanhross/lipydomics
  repo_url: https://github.com/dylanhross/lipydomics
  tools:
  - LiPydomics
  - Python
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3954
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# median-relative-error-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the median relative error (MDRE) of collision cross section (CCS) values against reference values after applying linear corrections — here, negative-mode TIMS CCS values from Vasilopoulou et al. and Tsugawa et al. calibrated toward DT reference values, reproducing the reported MDRE of −0.36%.

## When to use

Use when a collection of corrected/predicted CCS measurements with reference values needs its central accuracy quantified as a median relative error (MDRE), or when validating a calibration by reproducing a reported MDRE.

## When NOT to use

- The CCS values are already calibrated toward DT reference values and no linear correction from Figure S2 needs to be applied — a plain relative-error summary suffices without the calibration step
- The measurements are positive-mode or drift-gas conditions other than TIMS with the negative-mode linear corrections; the Figure S2 equations would not be applicable
- The goal is CCS prediction itself (e.g., training ML prediction models for the 22/23 lipid classes) rather than validating corrected measurements against DT references

## Inputs

- Negative-mode TIMS CCS measurements from the Vasilopoulou et al. constituent dataset
- Negative-mode TIMS CCS measurements from the Tsugawa et al. constituent dataset
- DT reference CCS values corresponding to each measurement
- Linear correction equations reported in Figure S2, one per constituent dataset

## Outputs

- corrected_ccs_table.csv containing a corrected CCS entry for every input measurement
- Computed post-correction MDRE (target: −0.36%) recorded alongside the reported value for comparison

## How to apply

Load the constituent CCS measurement datasets (e.g., the negative-mode TIMS CCS measurements from Vasilopoulou et al. and Tsugawa et al.) into pandas DataFrames. For each dataset, select its reported linear correction equation (Figure S2 in the source work) and apply it to the CCS values to calibrate them toward DT reference values. Concatenate the corrected CCS values from both datasets into a single table, then compute the relative error of each corrected CCS value against its DT reference and take the median — the post-correction MDRE (reported: −0.36%). Persist the corrected CCS table to corrected_ccs_table.csv with the computed MDRE recorded alongside the reported value so they can be compared. SciPy can support any fitting/regression steps, and LiPydomics/Python provide the surrounding analysis environment.

## Related tools

- **LiPydomics** (Python package providing the lipidomics analysis environment within which the CCS correction and MDRE computation are performed) — https://github.com/dylanhross/lipydomics
- **pandas** (Loads the constituent CCS datasets into DataFrames, concatenates corrected values into a single table, and computes the median relative error)
- **SciPy** (Supports the numerical/scientific computing routines underlying the correction and error calculation)

## Examples

```
import pandas as pd; df['rel_err'] = (df['ccs_corrected'] - df['ccs_dt']) / df['ccs_dt']; mdre = df['rel_err'].median(); df.to_csv('corrected_ccs_table.csv')
```

## Evaluation signals

- The computed MDRE must reproduce the reported post-correction value of −0.36% within reasonable numerical tolerance
- The corrected CCS table must contain a corrected CCS entry for every input measurement (no dropped rows after applying the linear corrections and concatenating both datasets)
- Each corrected CCS value must be traceable to its constituent dataset (Vasilopoulou et al. or Tsugawa et al.) and paired with its DT reference value
- The MDRE is a median, not a mean — a spot check on a few corrected values should confirm relative errors are computed per measurement as (corrected CCS − DT reference)/DT reference before taking the median

## Limitations

- The linear corrections are the specific equations reported in Figure S2 for the Vasilopoulou et al. and Tsugawa et al. datasets; they calibrate negative-mode TIMS CCS toward DT values and should not be applied to other datasets or modes without revalidation
- The skill validates accuracy against DT reference values only; agreement with other CCS reference scales (e.g., TWCCS) is not established by this computation
- The surrounding LiPydomics package is designed for standard lipidomics analysis protocols and is extensible as more data becomes available, but its experimental database and prediction models cover specific lipid classes (45 classes, 22–23 with models), so CCS values outside that coverage need external handling

## Evidence

- [supplementary] MDRE target value: [evidence span withheld — anchor status: not_found]
