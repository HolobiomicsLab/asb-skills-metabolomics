---
name: tims-ccs-linear-calibration-toward-dt-reference
description: Use when TIMS-derived CCS values (e.g., from Vasilopoulou et al. or Tsugawa
  et al. negative-mode datasets) must be calibrated toward DT reference CCS values
  and the residual error quantified as MDRE.
when_to_use_negative:
- Input CCS values are already DT-measured references rather than TIMS-derived values
  requiring calibration.
- The source dataset lacks a published linear correction equation (no Figure S2-style
  calibration available), so a linear correction cannot be justified.
- The analysis target is CCS prediction via machine learning models (e.g., DeepCCS
  or LipidCCS) rather than calibration of experimental TIMS measurements.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: LiPydomics
  role: Python package providing the lipidomics data-analysis environment (stats/data
    modules) in which the corrected CCS tables and error statistics are computed
  repo: https://github.com/dylanhross/lipydomics
- name: pandas
  role: Used to load the negative-mode TIMS CCS measurements into DataFrames and to
    concatenate corrected values from both datasets into a single table
- name: SciPy
  role: Scientific computing routines supporting the correction and error computations
- name: Python
  role: Implementation language for the correction and MDRE workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:ef8341100dfa9181668f13c44b5280c98837b842fe7d3c93057a58e96b0c420c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tims-ccs-linear-calibration-toward-dt-reference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies dataset-specific linear correction equations to negative-mode TIMS CCS values from Vasilopoulou et al. and Tsugawa et al. to calibrate them toward drift-tube (DT) reference values, then computes the post-correction median relative error (MDRE).

## When to use

Use when TIMS-derived CCS values (e.g., from Vasilopoulou et al. or Tsugawa et al. negative-mode datasets) must be calibrated toward DT reference CCS values and the residual error quantified as MDRE.

## When NOT to use

- Input CCS values are already DT-measured references rather than TIMS-derived values requiring calibration.
- The source dataset lacks a published linear correction equation (no Figure S2-style calibration available), so a linear correction cannot be justified.
- The analysis target is CCS prediction via machine learning models (e.g., DeepCCS or LipidCCS) rather than calibration of experimental TIMS measurements.

## Inputs

- Negative-mode TIMS CCS measurements from Vasilopoulou et al. dataset
- Negative-mode TIMS CCS measurements from Tsugawa et al. dataset
- Linear correction equations from Figure S2 of the source publication
- DT reference CCS values for each measured ion

## Outputs

- corrected_ccs_table.csv containing a corrected CCS entry for every input measurement
- Computed post-correction MDRE (target: −0.36%) recorded alongside the reported value for comparison

## How to apply

Load the negative-mode TIMS CCS measurements from the Vasilopoulou et al. and Tsugawa et al. constituent datasets into pandas DataFrames. For each dataset, select the linear correction equation reported in Figure S2 of the source publication and apply it to the corresponding CCS values to calibrate them toward DT reference values. Concatenate the corrected CCS values from both datasets into a single table, then compute the post-correction median relative error (MDRE) of the corrected CCS values against their DT reference values. Validate by checking that the computed MDRE reproduces the reported value of −0.36% within reasonable numerical tolerance and that every input measurement has a corrected CCS entry.

## Related tools

- **LiPydomics** (Python package providing the lipidomics data-analysis environment (stats/data modules) in which the corrected CCS tables and error statistics are computed) — https://github.com/dylanhross/lipydomics
- **pandas** (Used to load the negative-mode TIMS CCS measurements into DataFrames and to concatenate corrected values from both datasets into a single table)
- **SciPy** (Scientific computing routines supporting the correction and error computations)
- **Python** (Implementation language for the correction and MDRE workflow)

## Examples

```
import pandas as pd; corrected = pd.concat([vas_df, tsu_df]); corrected.to_csv('corrected_ccs_table.csv'); mdre = (corrected['ccs_corrected'] - corrected['ccs_dt']).div(corrected['ccs_dt']).median()
```

## Evaluation signals

- Computed MDRE reproduces the reported post-correction value of −0.36% within reasonable numerical tolerance.
- corrected_ccs_table.csv contains a corrected CCS entry for every input measurement (no rows dropped during correction or concatenation).
- Each dataset's corrected values were generated with the specific Figure S2 linear equation assigned to that dataset, not a pooled or mismatched equation.
- MDRE is computed as the median relative error against DT reference values, not the mean, so the metric matches the reported definition.

## Limitations

- Validation depends on access to the exact Figure S2 linear correction equations; without them the −0.36% MDRE cannot be reproduced.
- The correction is linear and specific to negative-mode TIMS CCS toward DT references; it does not generalize to positive-mode data or other IM technologies without recalibration.
- Reasonable numerical tolerance must be defined by the practitioner; the source card does not specify an exact tolerance band for the −0.36% comparison.

## Evidence

- [other] After applying the linear corrections to negative-mode TIMS CCS values toward DT values, the MDRE for negative mode TIMS CCS values is −0.36%.: [evidence span withheld — anchor status: not_found]
- [supplementary] Select the linear correction equation reported in Figure S2 for each dataset and apply it to its corresponding CCS values to calibrate them toward DT reference values.: [evidence span withheld — anchor status: not_found]
- [other] the computed MDRE must reproduce the reported post-correction value of −0.36% within reasonable numerical tolerance: [evidence span withheld — anchor status: not_found]
