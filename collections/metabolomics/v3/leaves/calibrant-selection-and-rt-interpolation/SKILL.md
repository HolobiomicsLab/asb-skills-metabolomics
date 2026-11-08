---
name: calibrant-selection-and-rt-interpolation
description: Use when lipid RTs were measured under a modified HILIC gradient or a
  different column length (100, 50, or 30 mm Kinetex HILIC) than the LiPydomics reference
  database and must be mapped onto the reference RT scale.
when_to_use_negative:
- RTs were measured on the exact reference HILIC gradient/column, so measured and
  database RTs are already on the same scale and no calibration is needed
- Candidate calibrant lipids belong to lipid classes without reference HILIC retention
  times in the database (only 23 of 45 classes carry HILIC RTs)
- The goal is CCS prediction or lipid identification from m/z and CCS alone rather
  than retention-time scale harmonization
tools:
- name: LiPydomics
  role: Provides the reference lipid database (HILIC RTs for 23 of 45 lipid classes)
    and the plotting module used to render interpolated-vs-reference RT plots
  repo: https://github.com/dylanhross/lipydomics
- name: SciPy
  role: Supplies the piecewise linear interpolation used to map measured calibrant
    RTs onto reference RTs
- name: Python
  role: Implementation language for the calibration script/module and CSV export
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:ccc9d0168caa4790986d2bd990cd4c4f64fca2aab1cd1c0a315da8c32ab57d2f
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# calibrant-selection-and-rt-interpolation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates measured lipid retention times from modified HILIC gradients or columns onto the LiPydomics reference HILIC RT scale via piecewise linear interpolation between two to four calibrant lipids, so identification can proceed against the package's lipid database under non-reference chromatographic conditions.

## When to use

Use when lipid RTs were measured under a modified HILIC gradient or a different column length (100, 50, or 30 mm Kinetex HILIC) than the LiPydomics reference database and must be mapped onto the reference RT scale.

## When NOT to use

- RTs were measured on the exact reference HILIC gradient/column, so measured and database RTs are already on the same scale and no calibration is needed
- Candidate calibrant lipids belong to lipid classes without reference HILIC retention times in the database (only 23 of 45 classes carry HILIC RTs)
- The goal is CCS prediction or lipid identification from m/z and CCS alone rather than retention-time scale harmonization

## Inputs

- Measured RTs of 2-4 calibrant lipids acquired under the modified HILIC gradient/column
- Reference HILIC retention times from the LiPydomics lipid database (23 lipid classes carry HILIC RTs)
- Measured RTs of non-calibrant lipid features to be mapped to the reference scale

## Outputs

- Piecewise linear RT calibration mapping (calibrant measured RT -> reference RT)
- Table of interpolated reference-scale RTs for non-calibrant lipids (CSV export)
- Plot of interpolated vs measured reference retention times for non-calibrant lipids

## How to apply

Load the reference HILIC retention times from the LiPydomics lipid database — 23 of the 45 lipid classes carry HILIC RTs — and select two to four calibrant lipids from those classes that were measured under the modified gradient/column. Implement piecewise (segment-wise) linear interpolation that maps each calibrant lipid's measured RT to its reference database RT, anchoring the calibration intervals at the calibrant points; SciPy provides the interpolation machinery and Python is the implementation language. Parameterize the routine per scenario, as the paper demonstrated across modified gradients on 100 mm, 50 mm, and 30 mm Kinetex HILIC columns. Apply the fitted calibration to measured RTs of non-calibrant lipids to obtain reference-scale RTs, then export the interpolated-vs-reference RT table (e.g. CSV) and generate a matplotlib-style plot via the package's plotting module. Validate by checking that interpolated RTs for non-calibrant lipids numerically and visually agree with measured reference RTs, with small residuals and overlapping interpolated lines across the demonstrated gradients and columns.

## Related tools

- **LiPydomics** (Provides the reference lipid database (HILIC RTs for 23 of 45 lipid classes) and the plotting module used to render interpolated-vs-reference RT plots) — https://github.com/dylanhross/lipydomics
- **SciPy** (Supplies the piecewise linear interpolation used to map measured calibrant RTs onto reference RTs)
- **Python** (Implementation language for the calibration script/module and CSV export)

## Examples

```
from scipy.interpolate import interp1d; cal_map = interp1d(measured_calib_rts, reference_calib_rts, kind='linear'); rt_ref = cal_map(measured_noncalib_rts)
```

## Evaluation signals

- Exactly two to four lipid calibrants with database reference RTs are selected per modified gradient/column condition
- Interpolated RTs for non-calibrant lipids show small residuals against their measured reference RTs, matching the paper's calibration evaluation
- Interpolated calibration lines overlap across the three demonstrated setups (100, 50, and 30 mm Kinetex HILIC columns)
- The exported RT table and plot of interpolated vs measured reference RTs are produced and consistent with each other

## Limitations

- Only 23 of the 45 lipid classes in the database contain HILIC retention times, restricting calibrant choice to those classes
- Calibration was demonstrated only for modified gradients on Kinetex HILIC columns of 100, 50, or 30 mm; other stationary phases or gradient regimes are not validated
- Piecewise linear interpolation does not extrapolate RTs outside the calibrant interval, so measured RTs beyond the outermost calibrants cannot be reliably mapped

## Evidence

- [other] The retention time calibration utility maps measured RTs to reference times via piecewise linear interpolation between calibrants: [evidence span withheld — anchor status: not_found]
- [abstract] we assembled a comprehensive experimental database of m/z and CCS of 45 lipid classes with 23 classes containing HILIC retention times: "we assembled a comprehensive experimental database of m/z and CCS of 45 lipid classes with 23 classes containing HILIC retention times"
- [readme] the `plotting` module has functions for generating various plots using computed statistics or the data itself: "the `plotting` module has functions for generating various plots using computed statistics or the data itself"
