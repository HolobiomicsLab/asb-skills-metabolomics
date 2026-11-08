---
name: retention-time-piecewise-linear-calibration
description: Use when lipid features were separated on a modified HILIC gradient or
  a non-reference Kinetex HILIC column (e.g. 50 mm or 30 mm instead of 100 mm), and
  their measured RTs must be mapped to the reference database RTs (23 lipid classes
  carry HILIC RTs) before identification.
when_to_use_negative:
- Data were acquired on the reference HILIC method and column, so RTs already sit
  on the reference scale and can be searched directly against the lipid database
- Fewer than two calibrant lipid classes were measured under the modified condition,
  so no piecewise interpolation interval can be defined
- The task is CCS prediction or statistical/multivariate analysis (stats module) rather
  than RT scale correction
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LiPydomics
  role: Provides the reference HILIC retention time database (23 of 45 lipid classes)
    and the plotting module used to render interpolated vs reference RT plots; calibration
    utility is exported as a package extension
  repo: https://github.com/dylanhross/lipydomics
- name: SciPy
  role: Supplies the piecewise linear interpolation used to map measured calibrant
    RTs to reference database RTs
- name: Python
  role: Implementation language for the calibration script/module and CSV export of
    the interpolated-vs-reference RT table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.0c02560
    title: 'LiPydomics: A Python Package for Comprehensive Prediction of Lipid Collision
      Cross Sections and Retention Times and Analysis of Ion Mobility-Mass Spectrometry-Based
      Lipidomics Data'
  source_content_hash: sha256:76dbfb62a09f5bfffff4d74654f7ec661b8915ba01148d52ab6c8383fc36103b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# retention-time-piecewise-linear-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates measured HILIC retention times from modified gradients or shorter columns onto the LiPydomics reference RT scale using piecewise linear interpolation between two to four calibrant lipid classes, so lipid features measured under non-reference HILIC conditions can be matched to the package's predicted/experimental lipid database.

## When to use

Use when lipid features were separated on a modified HILIC gradient or a non-reference Kinetex HILIC column (e.g. 50 mm or 30 mm instead of 100 mm), and their measured RTs must be mapped to the reference database RTs (23 lipid classes carry HILIC RTs) before identification.

## When NOT to use

- Data were acquired on the reference HILIC method and column, so RTs already sit on the reference scale and can be searched directly against the lipid database
- Fewer than two calibrant lipid classes were measured under the modified condition, so no piecewise interpolation interval can be defined
- The task is CCS prediction or statistical/multivariate analysis (stats module) rather than RT scale correction

## Inputs

- Measured HILIC retention times of calibrant lipid classes under the modified gradient or non-reference column (100, 50, or 30 mm Kinetex HILIC)
- Reference HILIC retention times for the same calibrant classes from the LiPydomics lipid database (23 classes with HILIC RTs)
- Measured RTs of non-calibrant lipid features to be mapped onto the reference scale

## Outputs

- Piecewise linear calibration mapping measured RT -> reference database RT for the chosen HILIC condition
- Interpolated reference-scale RT table for non-calibrant lipids (e.g. CSV export)
- Plot of interpolated vs measured reference retention times for the non-calibrant lipids (matplotlib-style, consistent with the LiPydomics plotting module)

## How to apply

Load the reference HILIC retention times from the LiPydomics lipid database, which covers 45 lipid classes with 23 classes containing HILIC RTs, and select two to four calibrant lipid classes that were actually measured under the modified gradient or column. Implement a piecewise linear interpolation (e.g. with SciPy) that maps each calibrant lipid's measured RT to its reference database RT across the calibration interval, and parameterize the routine per HILIC condition — the paper demonstrates three scenarios: modified gradients on 100 mm, 50 mm, and 30 mm Kinetex HILIC columns. Apply the fitted calibration to measured RTs of non-calibrant lipids to obtain interpolated reference-scale RTs, then plot interpolated vs measured reference RTs with the package's matplotlib-style plotting module. The rationale is that overlapping interpolated lines between calibrants demonstrated calibration accuracy across all three gradient/column combinations, so non-calibrant lipids can be assigned reference-scale RTs without re-measuring everything on the reference method.

## Related tools

- **LiPydomics** (Provides the reference HILIC retention time database (23 of 45 lipid classes) and the plotting module used to render interpolated vs reference RT plots; calibration utility is exported as a package extension) — https://github.com/dylanhross/lipydomics
- **SciPy** (Supplies the piecewise linear interpolation used to map measured calibrant RTs to reference database RTs)
- **Python** (Implementation language for the calibration script/module and CSV export of the interpolated-vs-reference RT table)

## Examples

```
python example_standard_analysis.py
```

## Evaluation signals

- Interpolated RTs of non-calibrant lipids agree with their measured reference RTs with small residual errors (visual and numeric agreement, as in the paper's calibration evaluation)
- Overlapping interpolated lines between adjacent calibrants demonstrate a consistent mapping across the full RT interval
- Calibration reproduces the paper's demonstrated accuracy for all three scenarios: modified gradients on 100 mm, 50 mm, and 30 mm Kinetex HILIC columns
- The exported interpolated-vs-reference RT table contains valid (monotonic, non-negative) reference-scale RTs for every non-calibrant lipid
- Calibrated RTs enable correct retrieval of the expected lipid class entries from the 145,388-entry predicted lipid database

## Limitations

- Calibration quality depends on using two to four lipid calibrants per condition; accuracy outside the calibrant-defined interpolation interval is not demonstrated
- Only three HILIC conditions were validated (modified gradients on 100, 50, and 30 mm Kinetex HILIC columns); other columns or gradients require new calibrant measurements
- Only the 23 lipid classes carrying HILIC RTs in the reference database can benefit from RT-based identification after calibration
- The package's 'missing signal' notes no separate code/data link for this specific calibration utility beyond the main repository, so the routine may need to be written against the public API

## Evidence
