---
name: la-icp-ms-calibration-curve-construction
description: Use when quantifying LA-ICP-MS elemental images from raw signal intensities,
  with calibration standards (e.g. gelatin) of known concentration available to derive
  per-isotope regression coefficients.
when_to_use_negative:
- Input data is already quantified concentration-based images — recalibration would
  double-correct signal
- No calibration standards of known concentration are available; the regression cannot
  be constructed without assigned standard values
- Performing drift correction or spike removal rather than intensity-to-concentration
  conversion — those are separate Pew2 tools (drift correction by polynomial fit,
  local mean/median filtering)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0153
tools:
- name: Pew2
  role: Provides the built-in standardization tool for assigning calibration values
    to rectangular image areas, fitting the regression, and applying the resulting
    calibration to sample images
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing import and export of LA-ICP-MS data underlying
    the calibration workflow, for batch/scripted processing
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Numerical backend used by Pew2 (Python-based implementation) to compute the
    line of best fit and validity coefficients
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:b8e5f11415e7c39216049372660c1d9ad371426f380303028467332b33c77c36
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# la-icp-ms-calibration-curve-construction

## Summary

Construct calibration curves from gelatin standards of known concentration to convert raw LA-ICP-MS signal intensities into quantified elemental images, as implemented in Pew2's built-in standardization tool. Regression per isotope yields intercept, gradient, r² and Sy,x coefficients used to transform sample signal data (e.g. 55Mn, 56Fe, 63Cu, 66Zn) into concentration-based images.

## When to use

Use when quantifying LA-ICP-MS elemental images from raw signal intensities, with calibration standards (e.g. gelatin) of known concentration available to derive per-isotope regression coefficients.

## When NOT to use

- Input data is already quantified concentration-based images — recalibration would double-correct signal
- No calibration standards of known concentration are available; the regression cannot be constructed without assigned standard values
- Performing drift correction or spike removal rather than intensity-to-concentration conversion — those are separate Pew2 tools (drift correction by polynomial fit, local mean/median filtering)

## Inputs

- Raw LA-ICP-MS data from vendor instruments (Agilent MassHunter batch folders .b directories, PerkinElmer Elan .xl, Thermo Qtegra/LDR CSV exports) containing calibration standard measurements
- Gelatin calibration standards with known elemental concentrations per calibration level
- Sample elemental signal data (e.g. 55Mn, 56Fe, 63Cu, 66Zn) acquired via LA-ICP-MS
- User-defined rectangular image areas within standard signals for each calibration level

## Outputs

- Calibration curve per isotope (line of best fit relating signal intensity to known concentration)
- Per-isotope calibration coefficients (intercept, gradient, r², Sy,x) e.g. calibration_coefficients_table.csv
- Quantified (concentration-based) elemental images derived from raw signal
- Re-usable saved calibration files applicable to other open images

## How to apply

Import raw vendor data (Agilent MassHunter batch folders, PerkinElmer Elan, Thermo Qtegra) via drag-and-drop into Pew2, then load the imported gelatin calibration standards into the built-in standardization tool. For each calibration level, assign a rectangular image area to the specific known calibration value; the software then calculates a line of best fit and validity coefficients from these assigned values, with optional weighting of the regression to improve accuracy at the lower end of the calibration range. Plot mean signal intensity per isotope against known standard concentrations to generate the calibration curve, yielding an intercept and gradient per isotope (e.g. 55Mn intercept 7.06, gradient 29.20, r² 0.9931). Apply the resulting calibration to the sample images to convert raw signal into quantified elemental images, and export the coefficients (intercept, gradient, r², Sy,x) as a calibration_coefficients_table.csv. Calibrations can be saved, applied to other open images, or exported for external use.

## Related tools

- **Pew2** (Provides the built-in standardization tool for assigning calibration values to rectangular image areas, fitting the regression, and applying the resulting calibration to sample images) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing import and export of LA-ICP-MS data underlying the calibration workflow, for batch/scripted processing) — https://github.com/djdt/pewlib
- **NumPy** (Numerical backend used by Pew2 (Python-based implementation) to compute the line of best fit and validity coefficients)

## Examples

```
pewpew  # open GUI, drag-and-drop Agilent batch folder, load gelatin standards into standardization tool, assign rectangular areas to known concentrations, fit and export calibration_coefficients_table.csv
```

## Evaluation signals

- Fitted linear regression per isotope should yield calibration coefficients comparable to published values (e.g. 55Mn intercept 7.06, gradient 29.20, r² 0.9931; 56Fe intercept 11.41, gradient 18.71, r² 0.9959; 63Cu r² 0.9940; 66Zn r² 0.9915), indicating a suitable line of best fit
- r² should approach 1 (published examples ≥ 0.9915) for the per-isotope intensity-vs-concentration regression
- Recorded coefficients table should contain all four values per isotope: intercept, gradient, r² and Sy,x
- Quantified elemental image should show accurate scale and color bars depicting laser and calibration parameters (as in the 66Zn example in Figure 4)
- Calibration must be exportable/saved and reproducible when applied to other open images

## Limitations

- Accuracy at the lower end of the calibration range depends on optional weighting of the regression; unweighted fits may be less accurate at low concentrations
- Spot-wise import requires at least one element to be present across the entirety of the sample and for peak positions of each element to be similar for calibration to be guided correctly
- Calibration quality is tied to the standards-preparation protocol (gelatin standards prepared as per Westerhausen et al.) — deviations in standard preparation propagate into the coefficients
- Non-vendor generic data must come in as flexible text image (2D delimited text) or file-per-line exports; arbitrary binary formats are not directly importable

## Evidence

- [methods] Pew2's calibration tool works by having the user assign rectangular image areas to specific known calibration values; the software then calculates a line of best fit and validity coefficients from these assigned values: [evidence span withheld — anchor status: not_found]
- [methods] optional weighting of the regression to improve accuracy at the lower end of the calibration range: [evidence span withheld — anchor status: not_found]
- [results] Drag-and-drop import of Agilent MassHunter batch folders supported directly by Pew2: [evidence span withheld — anchor status: external_doc]
- [methods] Calibrations can be saved, applied to other open images, or exported for external use: [evidence span withheld — anchor status: not_found]
