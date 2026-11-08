---
name: ccs-calibration-curve-fitting
description: Use when IMS-MS feature tables contain calibrant ions with reference
  CCS values and you need a fitted calibration function (linear, polynomial, or linearized
  power) to convert arrival times into CCS for unknown analytes.
when_to_use_negative:
- Stepped-field DTIMS experiments, where CCS can be computed directly from the fundamental
  low-field ion mobility equation rather than by fitting a calibration curve
- Data without calibrant ions of known reference CCS values — the regression requires
  reference compounds such as Agilent tune-mix
- Input data lacking arrival-time measurements (e.g. only m/z-intensity feature tables
  without drift/arrival times)
tools:
- name: AutoCCS
  role: Fits the CCS calibration curves (linear, polynomial, linearized power) from
    calibrant feature tables and reference CCS values via CLI flags --degree, --calib_method
    and --ppm
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python 3.7
  role: Runtime environment for AutoCCS; calibrant tables are loaded with pandas before
    least-squares regression
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:b3516b6c5cea05f00f47a03587c5222f5b5716222f075dd8155a3502959c7236
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - Python 3.7
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ccs-calibration-curve-fitting

## Summary

Fits CCS calibration curves with AutoCCS by regressing known calibrant CCS values (e.g. Agilent tune-mix) against arrival times: linear or polynomial functions for single-field DTIMS and TWIMS, and a linearized power function (Ruotolo et al. 2008) for traveling-wave platforms.

## When to use

Use when IMS-MS feature tables contain calibrant ions with reference CCS values and you need a fitted calibration function (linear, polynomial, or linearized power) to convert arrival times into CCS for unknown analytes.

## When NOT to use

- Stepped-field DTIMS experiments, where CCS can be computed directly from the fundamental low-field ion mobility equation rather than by fitting a calibration curve
- Data without calibrant ions of known reference CCS values — the regression requires reference compounds such as Agilent tune-mix
- Input data lacking arrival-time measurements (e.g. only m/z-intensity feature tables without drift/arrival times)

## Inputs

- Calibrant arrival-time/CCS reference table (e.g. TuneMix-CCS.txt from MassIVE MSV000085979)
- Calibrant feature tables (Features_csv/*.csv) from the IMS-MS platform being calibrated
- Frame metadata files (ImsMetadata/*.txt) and autoCCS_config.xml
- Sample metadata CSV with sample type, raw file name and ionization polarity columns

## Outputs

- calibration_coefficients.csv with fitted coefficients per ionization polarity and platform (polynomial degree terms, or power-function exponent/prefactor plus time offset)
- CCS values for unknown analyte ions computed with the fitted calibration function
- Run log (e.g. LogFiles/slim.log or single.log)

## How to apply

Load the calibrant arrival-time/CCS reference table (e.g. TuneMix-CCS.txt from MassIVE MSV000085979) plus the calibrant feature tables, then select the calibration function by platform: a degree-1 linear fit for single-field RapidFire-DTIMS (with reduced-CCS Ω' and P/T-adjusted Ω'' corrections using charge state, ion mass and buffer gas mass), a polynomial (quadratic or cubic via --degree with --calib_method poly) for TWIMS, or a linearized power function (--calib_method power, Eqs. 6-9 per Ruotolo et al. 2008). For TWIMS platforms first apply the platform-specific arrival-time adjustment: subtract the accumulation/access time (tacc) for SLIM-based IMS or apply the EDC delay coefficient for Waters Synapt data. Fit per ionization polarity and platform, writing coefficients to calibration_coefficients.csv (slope, intercept, R² for linear; exponent, pre-exponential coefficient and time offset for power) and logging to LogFiles (e.g. slim.log). Match calibrants using the ppm tolerance flag (e.g. --ppm 150 in the SLIM demo).

## Related tools

- **AutoCCS** (Fits the CCS calibration curves (linear, polynomial, linearized power) from calibrant feature tables and reference CCS values via CLI flags --degree, --calib_method and --ppm) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python 3.7** (Runtime environment for AutoCCS; calibrant tables are loaded with pandas before least-squares regression) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --degree 1 --mode single
```

## Evaluation signals

- calibration_coefficients.csv exists and contains a coefficient field for every fitted parameter (polynomial degree terms, or power-function exponent/prefactor plus time offset)
- For single-field DTIMS, slope, intercept, and R² of the least-squares fit are all reported in the calibration output
- Predicted calibrant CCS values agree with the reference tune-mix CCS values within the ppm tolerance used for calibrant matching (--ppm 150 in the SLIM demo)
- Separate coefficients are produced for each ionization polarity and platform present in the sample metadata
- The fit degree matches the requested --degree setting (e.g. degree 1 for single-field DTIMS, quadratic/cubic for TWIMS)

## Limitations

- CCS cannot be computed directly from the fundamental low-field ion mobility equation on TWIMS platforms (SLIM, Waters Synapt), so calibration depends entirely on the quality and coverage of the reference calibrant compounds
- Platform-specific arrival-time adjustments must be correct: accumulation/access time (tacc) subtraction for SLIM and the EDC delay coefficient for Waters Synapt; misapplied offsets bias the fitted function
- The paper reports no numeric R² threshold for accepting the calibration curve, so fit-quality checks are qualitative

## Evidence

- [methods] Calibration functions for single field and traveling wave-based methods — Linear function: "Calibration functions for single field and traveling wave-based methods
  - Linear function"
- [methods] Polynomial functions (e.g. quadratic or cubic functions): "Polynomial functions (e.g. quadratic or cubic functions)"
- [methods] Linearized power function [(Ruotolo,B.T. et al., 2008)]: "Linearized power function [(Ruotolo,B.T. et al., 2008)]"
- [abstract] linear calibration for Bruker timsTOF and non-linear calibration for traveling wave based-IMS in Waters Synapt and SLIM: [evidence span withheld — anchor status: not_found]
- [readme] python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml ... --degree 1: "python autoCCS.py
  --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml
  --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt""
- [readme] Demo dataset is publicly available at MassIVE (Accession: MSV000085979): "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
