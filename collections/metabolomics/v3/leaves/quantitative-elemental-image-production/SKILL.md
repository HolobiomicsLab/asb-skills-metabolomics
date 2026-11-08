---
name: quantitative-elemental-image-production
description: Reference for producing concentration-calibrated LA-ICP-MS images (e.g.
  55Mn, 56Fe, 63Cu, 66Zn in tissue sections) when gelatin standards with known concentrations
  are available and raw vendor batch data must be turned into quantified images.
when_to_use_negative:
- If the input data are already fully quantified elemental images or a calibration_coefficients_table.csv
  is already available — this skill is for producing them from raw signal.
- If no calibration standards of known concentration are available (e.g. spot-wise
  data lacking at least one element present across the entire sample with similar
  peak positions); Pew2 cannot fit a calibration without assigned known values.
- If the goal is solely image segmentation or RGB colocalization (e.g. k-means, Otsu,
  mean-thresholding) without converting intensity to concentration — those are separate
  steps after or independent of calibration.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Pew2
  role: GUI tool used to assign calibration regions to known standard concentrations,
    compute the line of best fit, and apply the resulting intercept and gradient to
    produce quantified elemental images
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library underpinning Pew2's import, processing and export of LA-ICP-MS
    data, enabling scripted batch conversion and calibration
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Numerical engine for fitting the linear-regression calibration curves and
    applying the gradient/intercept to the signal arrays
- name: PySide2
  role: Cross-platform Qt5 GUI layer of Pew2 in which the standardization/calibration
    tool is implemented
- name: Python
  role: Implementation language for Pew2 and pewlib, used for scripted calibration
    workflows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:c8bd003b30960292f077670152ddf5a0e2583b3f7ad6bb452097812c1cdf3056
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - PySide2
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
yes_rate: 0.9
n_claims: 10
---

# quantitative-elemental-image-production

## Summary

Converts raw LA-ICP-MS signal-intensity images into calibrated, concentration-quantified elemental images by fitting regression curves to gelatin standards of known concentration in Pew2, optionally weighting the fit to improve accuracy at the low end of the calibration range.

## When to use

Reference for producing concentration-calibrated LA-ICP-MS images (e.g. 55Mn, 56Fe, 63Cu, 66Zn in tissue sections) when gelatin standards with known concentrations are available and raw vendor batch data must be turned into quantified images.

## When NOT to use

- If the input data are already fully quantified elemental images or a calibration_coefficients_table.csv is already available — this skill is for producing them from raw signal.
- If no calibration standards of known concentration are available (e.g. spot-wise data lacking at least one element present across the entire sample with similar peak positions); Pew2 cannot fit a calibration without assigned known values.
- If the goal is solely image segmentation or RGB colocalization (e.g. k-means, Otsu, mean-thresholding) without converting intensity to concentration — those are separate steps after or independent of calibration.

## Inputs

- raw vendor LA-ICP-MS batch data (Agilent MassHunter G7200B+, Perkin Elmer Elan 'XL', Thermo Fischer Qtegra exports, delimited-text images)
- gelatin calibration standard LA-ICP-MS data with known per-isotope concentrations
- user-assigned rectangular regions of each standard image and their concentration values
- laser spot size and scan speed (entered manually after import to ensure correct image aspect and scale bar sizing)

## Outputs

- calibrated, concentration-quantified elemental images (e.g. 66Zn image with accurate scale and color bars depicting laser and calibration parameters)
- calibration_coefficients_table.csv with intercept, gradient, r² and Sy,x per isotope (e.g. 55Mn intercept 7.06, gradient 29.20, r² 0.9931, Sy,x 4.7; 56Fe intercept 11.41, gradient 18.71, r² 0.9959; 63Cu intercept 4.01, gradient 2.04, r² 0.9940; 66Zn intercept 8.48, gradient 4.12, r² 0.9915)

## How to apply

Load the sample LA-ICP-MS data (e.g. Agilent MassHunter batch folders, Perkin Elmer Elan, Thermo Qtegra exports) and the imported gelatin calibration standard data into Pew2. For each calibration level, define the rectangular region of the standard signal to use and record the known concentration of the standard, then plot the mean signal intensity for each isotope against those concentrations to generate the calibration curve. Fit a line of best fit (linear regression) per isotope, optionally weighting the regression to improve accuracy at the lower end of the calibration range, yielding an intercept and gradient for each element. Apply the fitted calibration (gradient and intercept) to the sample images, converting raw signal into quantified elemental images, and record the calibration coefficients (intercept, gradient, r², Sy,x) as a calibration_coefficients_table.csv. Rationale: Pew2 calculates the line of best fit and validity coefficients directly from the user-assigned calibration values, so accuracy depends on correctly defining standard regions and concentrations.

## Related tools

- **Pew2** (GUI tool used to assign calibration regions to known standard concentrations, compute the line of best fit, and apply the resulting intercept and gradient to produce quantified elemental images) — https://github.com/djdt/pewpew
- **pewlib** (Python library underpinning Pew2's import, processing and export of LA-ICP-MS data, enabling scripted batch conversion and calibration) — https://github.com/djdt/pewlib
- **NumPy** (Numerical engine for fitting the linear-regression calibration curves and applying the gradient/intercept to the signal arrays)
- **PySide2** (Cross-platform Qt5 GUI layer of Pew2 in which the standardization/calibration tool is implemented)
- **Python** (Implementation language for Pew2 and pewlib, used for scripted calibration workflows)

## Examples

```
pewpew  # open the imported Agilent batch, load gelatin standards into the built-in standardization tool, assign regions and concentrations, then apply the fitted calibration to the sample images
```

## Evaluation signals

- Verify the fitted linear regression produces calibration coefficients comparable to the published values (e.g. 55Mn intercept 7.06, gradient 29.20, r² 0.9931; 56Fe intercept 11.41, gradient 18.71, r² 0.9959; 63Cu intercept 4.01, gradient 2.04, r² 0.9940; 66Zn intercept 8.48, gradient 4.12, r² 0.9915).
- Check that the calibration_coefficients_table.csv contains intercept, gradient, r² and Sy,x for each isotope and that the exported coefficients match those displayed in Pew2.
- Confirm that the resulting quantified image's scale and color bars depict the laser and calibration parameters accurately (e.g. 66Zn image in Pew2).
- Inspect that optional weighting of the regression was applied where intended, since weighting can improve accuracy at the lower end of the calibration range.
- Cross-check that the calibration was applied to the sample images (not just the standard) and that the quantified images are in concentration units consistent with the standard values entered.

## Limitations

- Calibration requires at least one element to be present across the entirety of the sample with similar peak positions for each element when spot-wise data import is used (marker-element guided peak integration).
- Regions of extremely low or high signal from instrument noise may need selective filtering of instrumental artifacts, and drift correction (polynomial fit renormalization within a user-defined background slice) may need to be performed before calibration.
- The accuracy at the lower end of the calibration range depends on whether the optional regression weighting is applied; unweighted fits can be less accurate at low concentrations.
- Direct drag-and-drop import is limited to specific vendor formats (Agilent MassHunter G7200B+, Perkin Elmer Elan 'XL', Thermo Fischer Qtegra, Nu Vitesse, imzML, generic CSV); other vendors require flexible text-image or file-per-line imports.

## Evidence

- [supplementary] the software then calculates a line of best fit and validity coefficients from these assigned values, with optional weighting of the regression to improve accuracy at the lower end of the calibration range: [evidence span withheld — anchor status: not_found]
- [results] Table 2. Calibration Coefficients Calculated by the Software
Element Intercept Gradient r2 Sy,x
55Mn 7.06 29.20 0.9931 4.7
56Fe 11.41 18.71 0.9959 24.6
63Cu 4.01 2.04 0.9940 0.6
66Zn 8.48 4.12 0.9915 4.5: "Table 2. Calibration Coe fficienti cients Calculated by the Software
Element Intercept Gradient r2 S
y,x
55Mn 7.06 29.20 0.9931 4.7"
- [results] Spot-wise data import in Pew2 uses a single "marker" element to guide peak integration and requires at least one element to be present across the entire sample with similar peak positions for each element: [evidence span withheld — anchor status: not_found]
