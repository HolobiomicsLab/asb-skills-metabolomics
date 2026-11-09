---
name: calibration-residual-visualization
description: Use when HILIC retention times were measured under a modified gradient
  or a different column (e.g. 100, 50, or 30 mm Kinetex HILIC) than the reference
  database and must be mapped to the reference RT scale, with residuals checked for
  non-calibrant lipids.
when_to_use_negative:
- The lipid feature belongs to one of the lipid classes without reference HILIC retention
  times (only 23 of the 45 lipid classes carry HILIC RTs in the database)
- The chromatographic conditions match the reference database exactly, so no RT calibration
  transfer is needed
- The goal is CCS prediction or identification, which uses the package's CCS models
  and identification module rather than RT calibration
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: LiPydomics
  role: Provides the reference lipid database with HILIC RTs and the plotting module
    used to generate the interpolated-vs-reference RT figure
  repo: https://github.com/dylanhross/lipydomics
- name: SciPy
  role: Numerical library underlying Python implementation of the piecewise linear
    interpolation calibration
- name: matplotlib
  role: Matplotlib-style plotting consistent with the package's plotting module renders
    the interpolated vs measured reference RT plot
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:2549993feaf111f810d7fe39404e2956d220d1c19bb4ed971962efb45830559f
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
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# calibration-residual-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps measured HILIC retention times acquired under modified gradients or shorter columns onto a reference lipid RT scale by piecewise linear interpolation between calibrant lipids, then visualizes interpolated vs reference RTs for non-calibrant lipids to expose residuals. This validates that RT-based lipid identification in LiPydomics remains accurate across changed chromatographic conditions.

## When to use

Use when HILIC retention times were measured under a modified gradient or a different column (e.g. 100, 50, or 30 mm Kinetex HILIC) than the reference database and must be mapped to the reference RT scale, with residuals checked for non-calibrant lipids.

## When NOT to use

- The lipid feature belongs to one of the lipid classes without reference HILIC retention times (only 23 of the 45 lipid classes carry HILIC RTs in the database)
- The chromatographic conditions match the reference database exactly, so no RT calibration transfer is needed
- The goal is CCS prediction or identification, which uses the package's CCS models and identification module rather than RT calibration

## Inputs

- Measured HILIC retention times of 2-4 calibrant lipids acquired under the modified gradient/column
- Reference HILIC retention times from the LiPydomics lipid database (23 lipid classes)
- Measured HILIC retention times of non-calibrant lipid features to be mapped

## Outputs

- Piecewise linear RT calibration mapping for the given gradient/column condition
- Interpolated-vs-reference RT table for non-calibrant lipids (e.g. CSV export)
- Plot of interpolated vs measured reference retention times with residual errors for non-calibrant lipids

## How to apply

Load the reference HILIC retention times from the LiPydomics lipid database (23 of 45 lipid classes carry HILIC RTs) and select two to four calibrant lipids that were actually measured under the modified gradient or column. Implement piecewise linear interpolation mapping each calibrant's measured RT to its reference database RT, parameterized for the demonstrated scenarios (modified gradients on 100 mm, 50 mm, and 30 mm Kinetex HILIC columns). Apply the fitted mapping to measured RTs of non-calibrant lipids to obtain interpolated reference-scale RTs, then use the package's matplotlib-style plotting module to generate an interpolated-vs-reference RT plot with residuals. The rationale: overlapping interpolated lines across the three demonstrated conditions showed the piecewise approach transfers RTs between chromatographic setups, enabling RT-based Level 3 lipid identifications despite altered separation.

## Related tools

- **LiPydomics** (Provides the reference lipid database with HILIC RTs and the plotting module used to generate the interpolated-vs-reference RT figure) — https://github.com/dylanhross/lipydomics
- **SciPy** (Numerical library underlying Python implementation of the piecewise linear interpolation calibration)
- **matplotlib** (Matplotlib-style plotting consistent with the package's plotting module renders the interpolated vs measured reference RT plot)

## Examples

```
python example_standard_analysis.py
```

## Evaluation signals

- Calibrant lipids must map exactly: the piecewise interpolation passes through each calibrant's measured-to-reference RT pair
- Non-calibrant lipids show small residual errors between interpolated and their measured reference RTs, as in the paper's calibration evaluation
- Interpolated lines from different conditions (100, 50, 30 mm columns) overlap, demonstrating transfer accuracy
- Only lipids in classes with reference HILIC RTs (23 of 45 classes) should appear in the output table
- The plot is generated through the package's plotting module and the RT table exports cleanly as CSV

## Limitations

- Piecewise linear interpolation is only valid between calibrant RTs; extrapolation outside the calibration interval is not demonstrated
- Requires 2-4 measured calibrant lipids per modified condition, limiting use when few reference lipids are detected
- Accuracy was demonstrated only for modified gradients on 100, 50, and 30 mm Kinetex HILIC columns; other chemistries are untested
- The paper does not quantify a residual threshold; the stated check is visual plus numeric agreement of interpolated and reference RTs

## Evidence

- [abstract] 23 classes containing HILIC retention times: "we assembled a comprehensive experimental database of m/z and CCS of 45 lipid classes with 23 classes containing HILIC retention times"
- [readme] plotting module function: "the `plotting` module has functions for generating various plots using computed statistics or the data itself."
- [methods] package purpose: "`lipydomics` is a Python package for performing standard lipidomics analysis protocols on data in an efficient and reproducible fashion."
- [abstract] identification via HILIC RT: [evidence span withheld — anchor status: not_found]
