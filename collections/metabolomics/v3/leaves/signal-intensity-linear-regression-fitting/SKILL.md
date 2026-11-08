---
name: signal-intensity-linear-regression-fitting
description: Use when converting raw LA-ICP-MS elemental signal images (e.g. 55Mn,
  56Fe, 63Cu, 66Zn) into quantified concentration images using calibration standards
  of known concentration in Pew2 or pewlib.
when_to_use_negative:
- Input data are already quantified concentration images and no calibration against
  standards is needed
- The goal is only drift correction, spike removal, or ROI segmentation — those are
  preprocessing steps handled before calibration in Pew2
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Pew2
  role: GUI whose built-in standardization tool assigns known calibration values to
    rectangular standard regions, computes the line of best fit, and applies coefficients
    to sample images
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library for batch import/export and processing of LA-ICP-MS data that
    can be scripted for calibration workflows
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Numerical backend used by Pew2 for the Python-based regression and image computation
- name: Python
  role: Language in which Pew2 and pewlib are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:1dfb092ab1c955e9f5b89a24f2d192073181d56b5e470a7ae611e779c48d3b5d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# signal-intensity-linear-regression-fitting

## Summary

Fits a line of best fit (linear regression) between mean LA-ICP-MS signal intensities of gelatin standards and their known concentrations, yielding per-isotope intercept/gradient coefficients used to convert raw elemental signal images into quantified concentration images in Pew2.

## When to use

Use when converting raw LA-ICP-MS elemental signal images (e.g. 55Mn, 56Fe, 63Cu, 66Zn) into quantified concentration images using calibration standards of known concentration in Pew2 or pewlib.

## When NOT to use

- Input data are already quantified concentration images and no calibration against standards is needed
- The goal is only drift correction, spike removal, or ROI segmentation — those are preprocessing steps handled before calibration in Pew2

## Inputs

- Gelatin calibration standard signal data with known concentrations (per-isotope, e.g. 55Mn, 56Fe, 63Cu, 66Zn)
- Sample LA-ICP-MS elemental signal images (e.g. imported from Agilent MassHunter .b batch folders, PerkinElmer Elan .xl, Thermo Qtegra exports, or delimited text images)

## Outputs

- Per-isotope calibration coefficients table (intercept, gradient, r², Sy,x) exported as calibration_coefficients_table.csv
- Quantified elemental images (concentration-based) obtained by applying gradient and intercept to raw signal images

## How to apply

Import the gelatin calibration standard data (e.g. drag-and-drop Agilent MassHunter batch folders) alongside the sample signal data, then load the standards into Pew2's built-in standardization tool. For each calibration level, define the region of the standard signal to use and record the known concentration. Plot mean signal intensity per isotope against known concentrations and fit a linear regression, optionally weighting the regression to improve accuracy at the lower end of the calibration range. Record the intercept, gradient, r² and Sy,x per isotope (e.g. 55Mn intercept 7.06, gradient 29.20, r² 0.9931), export as calibration_coefficients_table.csv, and apply gradient/intercept to sample images to produce quantified elemental images.

## Related tools

- **Pew2** (GUI whose built-in standardization tool assigns known calibration values to rectangular standard regions, computes the line of best fit, and applies coefficients to sample images) — https://github.com/djdt/pewpew
- **pewlib** (Python library for batch import/export and processing of LA-ICP-MS data that can be scripted for calibration workflows) — https://github.com/djdt/pewlib
- **NumPy** (Numerical backend used by Pew2 for the Python-based regression and image computation)
- **Python** (Language in which Pew2 and pewlib are implemented)

## Examples

```
git clone https://github.com/djdt/pewpew; cd pewpew; pip install -r requirements.txt; pip install -e .; pewpew
```

## Evaluation signals

- Fitted r² per isotope should be comparable to published values (e.g. 55Mn 0.9931, 56Fe 0.9959, 63Cu 0.9940, 66Zn 0.9915), indicating a suitable line of best fit
- Calibration coefficients table contains intercept, gradient, r² and Sy,x columns per isotope (e.g. 55Mn intercept 7.06, gradient 29.20, Sy,x 4.7)
- Quantified images show accurate scale and color bars reflecting laser and calibration parameters (as in the 66Zn example)
- Applied calibration reproduces concentration-dependent trends matching the standard curve, with optional weighted regression improving accuracy at the low end of the range

## Limitations

- Requires standards spanning the expected concentration range; regression accuracy degrades if not weighted at the lower end of the calibration range
- Spot-wise import relies on at least one element present across the whole sample with similar peak positions for each element, which constrains which datasets can be calibrated this way
- Calibration is specific to the vendor formats Pew2 imports (Agilent MassHunter G7200B+, PerkinElmer Elan, Thermo Qtegra, delimited text); other formats need conversion

## Evidence

- [results] Line of best fit computed by Pew2: [evidence span withheld — anchor status: not_found]
- [results] Calibration coefficients reported in Table 2: [evidence span withheld — anchor status: not_found]
- [supplementary] Calibration tool usage listed in supplementary: [evidence span withheld — anchor status: not_found]
- [readme] pewlib for batch processing: [evidence span withheld — anchor status: external_doc]
