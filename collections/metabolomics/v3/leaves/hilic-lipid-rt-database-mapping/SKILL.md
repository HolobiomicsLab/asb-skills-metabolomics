---
name: hilic-lipid-rt-database-mapping
description: Use when HILIC-IM-MS lipidomics runs were acquired under a modified gradient
  or a different column length (100, 50, or 30 mm Kinetex HILIC) than the reference
  database, and measured lipid RTs must be rescaled to the reference RT scale for
  identification.
when_to_use_negative:
- Data were acquired under the same HILIC gradient and column as the reference database,
  so measured RTs already sit on the reference scale and no calibration is needed
- No calibrant lipids (2–4 required) were measured under the modified condition, leaving
  too few anchor points for the piecewise interpolation
- The lipid feature belongs to a class without HILIC RTs in the database (only 23
  of 45 lipid classes carry HILIC RTs)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LiPydomics
  role: Supplies the reference lipid database HILIC RTs (23 of 45 lipid classes),
    the plotting module for interpolated-vs-reference RT plots, and the package structure
    for the calibration utility
  repo: https://github.com/dylanhross/lipydomics
- name: SciPy
  role: Provides the piecewise linear interpolation used to map calibrant measured
    RTs to reference database RTs
- name: Python
  role: Language in which the calibration script/module is implemented and exported
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:ac2758ec5409d3829028f658aefa405a432e65c5c39f209153e279a085f975ee
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hilic-lipid-rt-database-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps measured HILIC retention times of lipids onto the LiPydomics reference lipid database RT scale using piecewise linear interpolation through 2–4 calibrant lipids, enabling measured RTs from modified HILIC gradients or columns (100, 50, or 30 mm Kinetex HILIC) to be compared against reference RTs covering 23 lipid classes.

## When to use

Use when HILIC-IM-MS lipidomics runs were acquired under a modified gradient or a different column length (100, 50, or 30 mm Kinetex HILIC) than the reference database, and measured lipid RTs must be rescaled to the reference RT scale for identification.

## When NOT to use

- Data were acquired under the same HILIC gradient and column as the reference database, so measured RTs already sit on the reference scale and no calibration is needed
- No calibrant lipids (2–4 required) were measured under the modified condition, leaving too few anchor points for the piecewise interpolation
- The lipid feature belongs to a class without HILIC RTs in the database (only 23 of 45 lipid classes carry HILIC RTs)

## Inputs

- Measured HILIC retention times of 2–4 calibrant lipids acquired under the modified gradient/column
- Measured HILIC RTs of non-calibrant lipid features to be rescaled
- Reference HILIC retention times from the LiPydomics lipid database (23 lipid classes carry HILIC RTs)

## Outputs

- Fitted piecewise linear RT calibration function mapping measured to reference RTs
- Table of interpolated (reference-scale) RTs vs measured RTs for non-calibrant lipids (e.g., CSV)
- Matplotlib-style plot of interpolated vs measured reference retention times

## How to apply

Load the reference HILIC retention times from the package lipid database (45 lipid classes total, 23 with HILIC RTs) and pick 2–4 calibrant lipids that were actually measured under the modified gradient/column condition. Build a piecewise linear interpolation (SciPy) that maps each calibrant lipid's measured RT to its reference database RT across the calibration interval, and parameterize the mapping for the demonstrated scenario (modified gradients on 100 mm, 50 mm, or 30 mm Kinetex HILIC columns). Apply the fitted interpolation to measured RTs of non-calibrant lipids to obtain reference-scale RTs, then plot interpolated vs measured reference RTs with the package's matplotlib-based plotting module and export the interpolated-vs-reference RT table as CSV. Acceptance is that interpolated RTs for non-calibrant lipids agree visually and numerically with their measured reference RTs, i.e., small residual errors as demonstrated in the paper's calibration evaluation across the three gradients/columns.

## Related tools

- **LiPydomics** (Supplies the reference lipid database HILIC RTs (23 of 45 lipid classes), the plotting module for interpolated-vs-reference RT plots, and the package structure for the calibration utility) — https://github.com/dylanhross/lipydomics
- **SciPy** (Provides the piecewise linear interpolation used to map calibrant measured RTs to reference database RTs)
- **Python** (Language in which the calibration script/module is implemented and exported)

## Examples

```
from scipy.interpolate import interp1d; rt_cal = interp1d(calibrant_measured_rt, calibrant_ref_rt, kind='linear', fill_value='extrapolate'); ref_rt = rt_cal(measured_rt)
```

## Evaluation signals

- Interpolated RTs for non-calibrant lipids show small residuals against their measured reference RTs (visual and numeric agreement, as in the paper's calibration evaluation)
- Calibration lines from overlapping interpolations across the three demonstrated conditions (100, 50, and 30 mm Kinetex HILIC columns) agree where they overlap
- Exactly 2–4 calibrant lipids are used per condition and each maps monotonically from measured to reference RT
- Exported CSV contains interpolated-vs-reference RT pairs for every non-calibrant lipid evaluated

## Limitations

- Accuracy depends on having 2–4 calibrant lipids measured under the modified condition; the demonstrated accuracy holds for modified gradients on 100, 50, or 30 mm Kinetex HILIC columns and may not generalize to other chemistries
- Only 23 of the 45 lipid classes in the reference database carry HILIC retention times, so calibration and reference mapping are limited to those classes
- The database and prediction models are built to be extensible as more data becomes available, implying current coverage gaps

## Evidence
