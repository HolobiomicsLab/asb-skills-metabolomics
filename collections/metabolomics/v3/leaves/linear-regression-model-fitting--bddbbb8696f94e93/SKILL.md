---
name: linear-regression-model-fitting
description: 'Use when calibrating single-field DTIMS-MS data with external calibrants
  (e.g. Agilent tune-mix): feature tables provide measured arrival times and a reference
  CCS list (TuneMix-CCS.txt) provides known CCS values, and a linear Ω″-vs-time calibration
  curve is needed.'
when_to_use_negative:
- Stepped-field DTIMS data, where CCS is determined directly from multiple field strengths
  and no calibration curve is required
- Traveling-wave data (Waters Synapt, SLIM, Bruker timsTOF) where AutoCCS uses non-linear
  (polynomial or linearized power function) calibration instead of a degree-1 linear
  fit
- Data with no calibrant CCS reference file (e.g. TuneMix-CCS.txt), since external
  calibration requires known calibrant CCS values
edam_operation: http://edamontology.org/operation_0366
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: AutoCCS
  role: Implements the single-field (standard and enhanced) DTIMS calibration workflow,
    including the --degree 1 linear calibration function invoked via autoCCS.py
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Language in which AutoCCS is written; numpy.polyfit (degree 1) performs the
    least-squares regression of Ω″ versus corrected arrival time
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:ed96411cb3d8d27f59ff9116c46ca7266678189545b473c70ffe676f6f05705d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  edam_operation: http://edamontology.org/operation_0366
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# linear-regression-model-fitting

## Summary

Fits a degree-1 least-squares calibration line (numpy.polyfit) relating P/T-adjusted reduced CCS values (Ω″) to corrected arrival times from single-field DTIMS calibrant runs, as AutoCCS does for its enhanced single-field method (Eq. 4).

## When to use

Use when calibrating single-field DTIMS-MS data with external calibrants (e.g. Agilent tune-mix): feature tables provide measured arrival times and a reference CCS list (TuneMix-CCS.txt) provides known CCS values, and a linear Ω″-vs-time calibration curve is needed.

## When NOT to use

- Stepped-field DTIMS data, where CCS is determined directly from multiple field strengths and no calibration curve is required
- Traveling-wave data (Waters Synapt, SLIM, Bruker timsTOF) where AutoCCS uses non-linear (polynomial or linearized power function) calibration instead of a degree-1 linear fit
- Data with no calibrant CCS reference file (e.g. TuneMix-CCS.txt), since external calibration requires known calibrant CCS values

## Inputs

- Calibrant feature tables (Features_csv) from single-field RapidFire-DTIMS-MS runs
- Reference CCS list for calibrant ions (e.g. TuneMix-CCS.txt from MassIVE MSV000085979)
- Per-ion charge state z, ion mass mI, buffer gas mass mB (from autoCCS_config.xml / target list)
- Run pressure and temperature values for the P/T correction

## Outputs

- calibration_parameters.csv containing fitted slope, intercept, and R^2 of the linear Ω″-vs-arrival-time calibration curve
- Fitted degree-1 polynomial model used to convert sample arrival times to CCS values

## How to apply

Load the calibrant feature tables from the single-field RapidFire-DTIMS Features_csv directory and the reference CCS list TuneMix-CCS.txt from MassIVE MSV000085979 with pandas. For each calibrant ion, compute the reduced CCS Ω' = (1/z)·sqrt(mI/(mI+mB))·Ω using the ion charge state z, ion mass mI, and buffer gas mass mB, then apply the pressure/temperature correction Ω″ = (P/T)·Ω' (Eq. 3). Fit a degree-1 polynomial of Ω″ versus corrected arrival time with numpy.polyfit (matching AutoCCS's --degree 1 setting) by least-squares regression. Compute the coefficient of determination (R^2) and write the fitted slope, intercept, and R^2 to calibration_parameters.csv; accept the curve if the fit converges and R^2 is reported alongside slope and intercept (the paper defines no numeric R^2 threshold, so a qualitative high-fit check applies).

## Related tools

- **AutoCCS** (Implements the single-field (standard and enhanced) DTIMS calibration workflow, including the --degree 1 linear calibration function invoked via autoCCS.py) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Language in which AutoCCS is written; numpy.polyfit (degree 1) performs the least-squares regression of Ω″ versus corrected arrival time) — https://www.python.org/downloads/

## Examples

```
python autoCCS.py --feature_files "data/SingleField-RapidFire-DTIMS/Features_csv/*.csv" --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --degree 1 --mode single
```

## Evaluation signals

- The degree-1 least-squares fit converges without error from numpy.polyfit
- calibration_parameters.csv contains all three fields: slope, intercept, and R^2
- Ω″ values reproduce the two-stage transformation: reduced CCS Ω' = (1/z)·sqrt(mI/(mI+mB))·Ω followed by the (P/T) scaling
- Calibrant points fall close to the fitted line (qualitative high-fit check, since the paper reports no numeric R^2 threshold)
- Fitted calibration applied to calibrants' own arrival times recovers the TuneMix-CCS.txt reference CCS values

## Limitations

- The paper reports no numeric R^2 threshold, so fit quality is only judged qualitatively
- Requires charge state, ion mass, buffer gas mass, and known CCS for every calibrant ion; missing values prevent computing Ω″
- The enhanced single-field P/T correction assumes calibrant and sample runs may differ in pressure and temperature, so metadata (framemeta files) must carry those conditions
- Discussion section content is absent from the document; limitations beyond those stated cannot be extracted

## Evidence

- [abstract] AutoCCS provides accurate and reproducible CCS determination for standard and unknown analyte ions across IMS-MS platforms, field methods, ionization modes and collision gases: [evidence span withheld — anchor status: not_found]
- [methods] Calibration with polynomial functions: "Polynomial functions (e.g. quadratic or cubic functions)"
- [readme] Linear calibration function support: "Calibration functions for single field and traveling wave-based methods
  - Linear function"
- [readme] Degree-1 CLI setting for single-field mode: "--degree 1 --single_mode batch --mode single"
