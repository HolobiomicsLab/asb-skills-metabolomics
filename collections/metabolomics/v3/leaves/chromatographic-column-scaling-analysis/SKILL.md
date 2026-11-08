---
name: chromatographic-column-scaling-analysis
description: Use when measured HILIC RTs come from a modified gradient or a shorter
  column (100/50/30 mm Kinetex HILIC) and must be mapped onto the reference database
  RT scale before lipid identification by retention time.
when_to_use_negative:
- The chromatography already matches the reference database conditions, so measured
  RTs can be compared directly without scaling
- The lipid class of interest is one of the 22+ classes lacking HILIC retention times
  in the database, so no calibrant or reference RT exists
- The target analyte's RT falls outside the interval spanned by the two to four calibrant
  lipids, where piecewise linear interpolation is undefined
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LiPydomics
  role: Supplies the reference lipid database (HILIC RTs for 23 of 45 lipid classes)
    and the matplotlib-style plotting module used to compare interpolated vs reference
    RTs
  repo: https://github.com/dylanhross/lipydomics
- name: SciPy
  role: Provides the piecewise linear interpolation routine used to map measured calibrant
    RTs to reference database RTs
- name: Python
  role: Implementation language for the calibration utility, exported as a script/module
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:916cc856afe8adfafb05225284dd68a63c2bbbaa6e732c54031b1670efd225a8
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chromatographic-column-scaling-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates HILIC retention times measured under modified gradients or shorter Kinetex HILIC columns (100, 50, or 30 mm) onto the LiPydomics reference lipid-database RT scale using piecewise linear interpolation through two to four lipid calibrants.

## When to use

Use when measured HILIC RTs come from a modified gradient or a shorter column (100/50/30 mm Kinetex HILIC) and must be mapped onto the reference database RT scale before lipid identification by retention time.

## When NOT to use

- The chromatography already matches the reference database conditions, so measured RTs can be compared directly without scaling
- The lipid class of interest is one of the 22+ classes lacking HILIC retention times in the database, so no calibrant or reference RT exists
- The target analyte's RT falls outside the interval spanned by the two to four calibrant lipids, where piecewise linear interpolation is undefined

## Inputs

- Measured HILIC retention times for two to four calibrant lipids under the modified gradient/column
- Reference HILIC retention times for the calibrant lipids from the LiPydomics lipid database (23 lipid classes carry HILIC RTs)
- Measured HILIC RTs of non-calibrant lipids to be mapped to the reference scale

## Outputs

- Piecewise linear RT calibration function parameterized per column length (100, 50, or 30 mm)
- Reference-scale (interpolated) retention times for non-calibrant lipids
- Interpolated-vs-reference RT table (e.g., CSV)
- Plot of interpolated vs measured reference retention times for non-calibrant lipids

## How to apply

Load the reference HILIC retention times from the LiPydomics lipid database (23 of 45 lipid classes carry HILIC RTs) and pick two to four calibrant lipids measured under the modified gradient/column. Fit a piecewise linear interpolation that maps each calibrant's measured RT to its reference database RT, parameterized for the demonstrated scenarios (modified gradients on 100 mm, 50 mm, and 30 mm Kinetex HILIC columns). Apply the fitted calibration to measured RTs of non-calibrant lipids to obtain reference-scale RTs, then plot interpolated vs measured reference RTs for the non-calibrants using the package's matplotlib-style plotting module and export the interpolated-vs-reference RT table (e.g., CSV). The rationale is that overlapping interpolated lines between calibrants demonstrated accuracy across all three column lengths in the paper's calibration evaluation.

## Related tools

- **LiPydomics** (Supplies the reference lipid database (HILIC RTs for 23 of 45 lipid classes) and the matplotlib-style plotting module used to compare interpolated vs reference RTs) — https://github.com/dylanhross/lipydomics
- **SciPy** (Provides the piecewise linear interpolation routine used to map measured calibrant RTs to reference database RTs)
- **Python** (Implementation language for the calibration utility, exported as a script/module)

## Examples

```
from scipy.interpolate import interp1d; rt_ref = interp1d(rt_meas_calibrants, rt_ref_db_calibrants, kind='linear')(rt_meas_noncalibrants)
```

## Evaluation signals

- Interpolated RTs for non-calibrant lipids show small residual errors against their measured reference RTs, the paper's acceptance check
- Interpolated lines from adjacent calibrants overlap across the calibration interval, as demonstrated for all three column lengths
- The calibration uses two to four calibrant lipids per condition, within the range demonstrated across three gradients and columns of 100, 50, or 30 mm
- Only lipid classes with HILIC RTs in the database (23 of 45) can serve as calibrants; using a class without reference RTs is a setup error
- The exported CSV table and plot agree numerically and visually on interpolated vs reference RTs

## Limitations

- Calibration was demonstrated specifically on Kinetex HILIC columns of 100, 50, and 30 mm; other column chemistries or dimensions are not validated
- Piecewise linear interpolation cannot extrapolate beyond the RT interval spanned by the chosen calibrants
- Only 23 of 45 lipid classes in the database contain HILIC retention times, limiting calibrant and RT-based identification coverage
- Accuracy rests on the demonstrated scenario of two to four calibrants; fewer calibrants may degrade the piecewise fit

## Evidence
