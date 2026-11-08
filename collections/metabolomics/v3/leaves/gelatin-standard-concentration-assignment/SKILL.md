---
name: gelatin-standard-concentration-assignment
description: 'Decision support for quantifying LA-ICP-MS elemental images against
  gelatin calibration standards: when raw isotope signal maps (55Mn, 56Fe, 63Cu, 66Zn)
  plus a standard of known concentrations must be converted to μg/g concentration
  images.'
when_to_use_negative:
- Data are already concentration-calibrated (e.g. a precalibrated NumPy export) —
  recalibration would be redundant
- No standard of known concentration is available; drift correction or spike removal
  alone cannot produce quantitative values
- Signal derives from a non-gelatin matrix whose certified reference values differ;
  the concentration assignments must match the actual standard used
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Pew2
  role: GUI application whose built-in standardization tool assigns standard regions
    to known concentrations and fits the calibration regression
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library enabling scripted batch calibration of LA-ICP-MS data outside
    the GUI
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Underlying numeric library used by Pew2 for the regression and image calculations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:d07fc84f0f42920a15b58e33c99b07ecc444715636312dece456091f56276a52
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gelatin-standard-concentration-assignment

## Summary

Assign rectangular image regions of a LA-ICP-MS gelatin standard acquisition to their known elemental concentrations and fit a weighted linear regression per isotope, producing intercept/gradient/r² calibration coefficients (e.g. 55Mn intercept 7.06, gradient 29.20, r² 0.9931) that convert raw signal images into quantified elemental maps in Pew2.

## When to use

Decision support for quantifying LA-ICP-MS elemental images against gelatin calibration standards: when raw isotope signal maps (55Mn, 56Fe, 63Cu, 66Zn) plus a standard of known concentrations must be converted to μg/g concentration images.

## When NOT to use

- Data are already concentration-calibrated (e.g. a precalibrated NumPy export) — recalibration would be redundant
- No standard of known concentration is available; drift correction or spike removal alone cannot produce quantitative values
- Signal derives from a non-gelatin matrix whose certified reference values differ; the concentration assignments must match the actual standard used

## Inputs

- Raw LA-ICP-MS batch data for gelatin calibration standards (Agilent MassHunter .b folder, Perkin Elmer Elan .xl, Thermo Qtegra export, or CSV text image)
- Known elemental concentrations for each gelatin calibration level
- Sample elemental signal images (e.g. 55Mn, 56Fe, 63Cu, 66Zn) awaiting calibration

## Outputs

- Per-isotope calibration coefficients (intercept, gradient, r², Sy,x) — e.g. 55Mn 7.06/29.20/0.9931
- Calibrated concentration-based elemental images of the sample
- Exportable calibration file reusable on other open images or externally

## How to apply

Import the raw Agilent MassHunter batch (or other vendor export) for the gelatin standard into Pew2 via drag-and-drop, then load it into the built-in standardization tool. For each calibration level, define the rectangular region of the standard signal to use and record the known concentration of that gelatin level. Pew2 plots mean isotope intensity (e.g. 55Mn, 56Fe, 63Cu, 66Zn) against known concentration, fits a line of best fit per isotope, and offers optional regression weighting to improve accuracy at the lower end of the calibration range. Check the fitted coefficients (intercept, gradient, r², Sy,x) against expected values such as 55Mn intercept 7.06, gradient 29.20, r² 0.9931 before applying them to the sample images; calibrations can be saved, applied to other open images, or exported for external use.

## Related tools

- **Pew2** (GUI application whose built-in standardization tool assigns standard regions to known concentrations and fits the calibration regression) — https://github.com/djdt/pewpew
- **pewlib** (Python library enabling scripted batch calibration of LA-ICP-MS data outside the GUI) — https://github.com/djdt/pewlib
- **NumPy** (Underlying numeric library used by Pew2 for the regression and image calculations)

## Examples

```
pewpew  # drag-and-drop the Agilent MassHunter .b folder of the gelatin standard, load into the standardization tool, assign regions to known concentrations, and export calibration_coefficients_table.csv
```

## Evaluation signals

- Fitted r² per isotope approaches published values (55Mn 0.9931, 56Fe 0.9959, 63Cu 0.9940, 66Zn 0.9915)
- Calibration coefficients match the article's Table 2 within reasonable tolerance (e.g. 55Mn intercept 7.06, gradient 29.20)
- Calibrated scale bar on the 66Zn image reflects correct laser and calibration parameters (0.25 s acquisition, 10 μm spot, 40 μm/s scan)
- Calibration exported file applies without error to other open sample images in Pew2

## Limitations

- Spot-wise import requires at least one 'marker' element present across the entire sample with similar peak positions across elements
- Quantification accuracy depends on gelatin standards prepared as per Westerhausen et al.; deviations in standard preparation shift the calibration
- Optional regression weighting is needed to improve accuracy at the low end of the concentration range; unweighted fits may underperform there

## Evidence

- [other] Pew2's calibration tool works by having the user assign rectangular image areas to specific known calibration values: [evidence span withheld — anchor status: not_found]
- [methods] Quantification of the tissue was performed using gelatin calibration standards prepared as per Westerhausen et al. 30: [evidence span withheld — anchor status: not_found]
- [results] Calibration coefficients table (Table 2): 55Mn intercept 7.06, gradient 29.20, r2 0.9931: [evidence span withheld — anchor status: not_found]
