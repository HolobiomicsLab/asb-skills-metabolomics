---
name: calibration-coefficient-reporting
description: Use when LA-ICP-MS signal data (Agilent MassHunter, Perkin Elmer Elan,
  Thermo Qtegra exports) must be converted into concentration-based elemental images
  using standards of known concentration, and the calibration regression coefficients
  need to be reported or exported for validation and reuse.
when_to_use_negative:
- The input data is already concentration-calibrated (e.g. a precalibrated NumPy export
  is provided) and no new standard-based regression is required.
- Only peak detection, drift correction, spike removal, or segmentation is needed
  without quantitative concentration conversion.
- No standards of known concentration are available to anchor the regression — the
  skill cannot produce valid intercept/gradient values without calibration levels.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: Pew2
  role: GUI (Python/NumPy/PySide2) that implements the standardization tool for assigning
    rectangular areas to known calibration values, fitting the line of best fit, and
    saving/exporting the calibration
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library for batch conversion and processing of LA-ICP-MS data, including
    import/export of vendor formats used by the calibration workflow
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Numerical backend used by Pew2 for the regression fitting and image arithmetic
    underlying the calibration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:c8c3882669c17c2a35ae51cdb98a5f78e697b38f91f929f96eaa54ff7d2ec29b
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
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# calibration-coefficient-reporting

## Summary

Fit linear intensity-versus-concentration regressions for each isotope (e.g. 55Mn, 56Fe, 63Cu, 66Zn) against gelatin calibration standards in Pew2's built-in standardization tool, then record and export the intercept, gradient, r² and Sy,x coefficients (e.g. 55Mn: intercept 7.06, gradient 29.20, r² 0.9931) so quantified elemental images are traceable and reproducible.

## When to use

Use when LA-ICP-MS signal data (Agilent MassHunter, Perkin Elmer Elan, Thermo Qtegra exports) must be converted into concentration-based elemental images using standards of known concentration, and the calibration regression coefficients need to be reported or exported for validation and reuse.

## When NOT to use

- The input data is already concentration-calibrated (e.g. a precalibrated NumPy export is provided) and no new standard-based regression is required.
- Only peak detection, drift correction, spike removal, or segmentation is needed without quantitative concentration conversion.
- No standards of known concentration are available to anchor the regression — the skill cannot produce valid intercept/gradient values without calibration levels.

## Inputs

- Raw LA-ICP-MS line-by-line data in vendor formats (Agilent MassHunter .b batch folders, Perkin Elmer Elan .xl, Thermo Qtegra CSV/LDR, CSV images)
- Gelatin calibration standard data with known concentrations for each calibration level (per Westerhausen et al.)
- User-defined rectangular ROI assignments on the standard image for each calibration point
- Calibration concentrations for the gelatin standard

## Outputs

- Per-isotope calibration coefficients (intercept, gradient, r², Sy,x), e.g. 55Mn: 7.06 / 29.20 / 0.9931 / 4.7; 56Fe: 11.41 / 18.71 / 0.9959 / 24.6; 63Cu: 4.01 / 2.04 / 0.9940 / 0.6; 66Zn: 8.48 / 4.12 / 0.9915 / 4.5
- calibration_coefficients_table.csv exported for external use
- Concentration-based quantified elemental images for each isotope
- Saved calibration object reusable on other open images

## How to apply

After drag-and-drop importing the raw vendor data (Agilent MassHunter G7200B+, Perkin Elmer Elan 'XL', or Thermo Qtegra exports) and loading the gelatin calibration standards into Pew2's built-in standardization tool, assign rectangular image areas to each calibration level and record the known concentration of the standard. For each isotope, compute the mean signal intensity per calibration level and fit a line of best fit (linear regression) of intensity versus concentration, optionally weighting the regression toward the lower end of the calibration range to improve accuracy at low concentrations. Apply the resulting gradient and intercept to the sample images (e.g. 66Zn, 55Mn, 56Fe, 63Cu) to convert raw signal into quantified elemental images. Record the per-isotope coefficients (intercept, gradient, r², Sy,x) in a calibration_coefficients_table.csv for validation against published values such as 56Fe intercept 11.41, gradient 18.71, r² 0.9959, Sy,x 24.6.

## Related tools

- **Pew2** (GUI (Python/NumPy/PySide2) that implements the standardization tool for assigning rectangular areas to known calibration values, fitting the line of best fit, and saving/exporting the calibration) — https://github.com/djdt/pewpew
- **pewlib** (Python library for batch conversion and processing of LA-ICP-MS data, including import/export of vendor formats used by the calibration workflow) — https://github.com/djdt/pewlib
- **NumPy** (Numerical backend used by Pew2 for the regression fitting and image arithmetic underlying the calibration)

## Examples

```
python -m pewpew  # then drag-and-drop the Agilent batch folders into the main window, assign ROIs to calibration values, and export calibration_coefficients_table.csv
```

## Evaluation signals

- Fitted linear regression for each isotope yields r² comparable to published values (e.g. 55Mn r² 0.9931, 56Fe r² 0.9959, 63Cu r² 0.9940, 66Zn r² 0.9915), indicating a suitable line of best fit for quantification.
- Recorded intercept, gradient, and Sy,x per isotope match the calibration_coefficients_table.csv (e.g. 55Mn intercept 7.06, gradient 29.20, Sy,x 4.7).
- Quantified elemental image (e.g. 66Zn) shows an accurate scale bar and color bar reflecting correct laser and calibration parameter entry.
- The saved calibration can be applied to another open image or exported without altering the fitted coefficients.
- Optionally weighted regression shows improved accuracy at the lower end of the calibration range relative to the unweighted fit.

## Limitations

- Calibration requires at least one element present across the entirety of the sample with similar peak positions across elements (single-marker guide for peak integration) — failure to satisfy this can produce misleading spot-wise calibration.
- The calibration tool requires the user to manually assign rectangular areas to known calibration values; a poorly defined ROI or mis-entered concentration propagates directly into the fitted coefficients.
- Laser spot size and speed must be entered manually after import; incorrect entries distort image aspect and scale-bar sizing, indirectly affecting coefficient interpretation.
- Only linear regression is applied; non-linear or multi-point calibration beyond a weighted linear fit is not described.
- Interoperability with non-supported vendors is limited to 2D delimited text images and file-per-line exports, which may lack calibration metadata.

## Evidence

- [results] assign rectangular image areas to specific known calibration values; the software then calculates a line of best fit and validity coefficients: [evidence span withheld — anchor status: not_found]
- [methods] Quanti fi cation of the tissue was performed using gelatin calibration standards prepared as per Westerhausen et al.: "Quanti fi cation of the tissue was performed using gelatin calibration standards prepared as per Westerhausen et al."
- [results] Table 2. Calibration Coe fficienti cients Calculated by the Software Element Intercept Gradient r2 Sy,x 55Mn 7.06 29.20 0.9931 4.7: "Table 2. Calibration Coe fficienti cients Calculated by the Software Element Intercept Gradient r2 S
y,x
55Mn 7.06 29.20 0.9931 4.7"
- [results] Image of 66Zn imported into Pew2. The scale and color bars depict accurate input of laser and calibration parameters: "Figure 4. Image of 66Zn imported into Pew2. The scale and color bars
depict accurate input of laser and calibration parameters."
- [readme] Pewlib is a library for importing, processing and exporting LA-ICP-MS data: [evidence span withheld — anchor status: external_doc]
- [supplementary] Record the calibration coefficients (intercept, gradient, r², and Sy,x) for each isotope and export them as a calibration_coefficients_table.csv file.: [evidence span withheld — anchor status: not_found]
