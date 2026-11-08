---
name: linearized-power-function-fitting
description: Use when calibrating TWIMS-MS arrival times to CCS (e.g. SLIM-based IMS
  or Waters Synapt) using reference calibrants such as Agilent tune-mix, and a power-law
  relationship (rather than polynomial) is desired for the calibration function.
when_to_use_negative:
- The data are stepped-field or single-field DTIMS, where CCS can be computed directly
  from the fundamental low-field ion mobility equation and no TWIMS calibration function
  is needed.
- A polynomial (quadratic/cubic) calibration function is preferred — use --calib_method
  poly instead of the power method.
- Input is a Bruker timsTOF dataset, which AutoCCS handles with linear calibration
  rather than the traveling-wave power function.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Implements the linearized power calibration (--calib_method power) for traveling
    wave-based IMS; invoked as the autoCCS.py command-line entry point
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python 3.7
  role: Runtime environment for installing dependencies via pip/conda and executing
    autoCCS.py
  repo: https://www.python.org/downloads/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:0c5bc662a79a73df03ba31626526f507bf1479113fda18c8d0dfc99377ce7e1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - Python 3.7
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# linearized-power-function-fitting

## Summary

Fits AutoCCS's linearized power (TWIMS) calibration function — log-log regression of known calibrant CCS values against arrival-time terms per Ruotolo et al. 2008 — for traveling-wave IMS platforms (Waters Synapt, SLIM) where CCS cannot be computed directly from the low-field mobility equation.

## When to use

Use when calibrating TWIMS-MS arrival times to CCS (e.g. SLIM-based IMS or Waters Synapt) using reference calibrants such as Agilent tune-mix, and a power-law relationship (rather than polynomial) is desired for the calibration function.

## When NOT to use

- The data are stepped-field or single-field DTIMS, where CCS can be computed directly from the fundamental low-field ion mobility equation and no TWIMS calibration function is needed.
- A polynomial (quadratic/cubic) calibration function is preferred — use --calib_method poly instead of the power method.
- Input is a Bruker timsTOF dataset, which AutoCCS handles with linear calibration rather than the traveling-wave power function.

## Inputs

- Calibrant reference table mapping arrival times to known CCS values (e.g. data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt style TuneMix-CCS file for Agilent tune-mix)
- Feature files (CSV) with detected ion arrival times for the calibrant runs
- Frame metadata files (ImsMetadata) and autoCCS_config.xml configuration file
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns

## Outputs

- calibration_coefficients.csv containing the fitted power-function exponent, pre-exponential coefficient, and time offset per ionization polarity and platform
- Log file (e.g. LogFiles/slim.log) recording the calibration run
- TWIMS CCS values for unknown analytes computed with the fitted power function

## How to apply

Load the calibrant arrival-time/CCS reference table (e.g. the TuneMix-CCS file for Agilent tune-mix from demo dataset MassIVE MSV000085979) together with the feature files into the AutoCCS Python 3.7 environment. First apply the platform-specific arrival-time adjustment for the traveling-wave-based method — subtract the accumulation/access time (tacc) for SLIM-based IMS and apply the EDC delay coefficient for Waters Synapt data. Then fit the linearized power function (Eqs. 6-9, after Ruotolo et al. 2008) by log-log linear regression with --calib_method power, jointly estimating the exponent, the pre-exponential coefficient, and the time offset. AutoCCS writes the fitted coefficients for each ionization polarity and platform to calibration_coefficients.csv, with run logs routed to LogFiles/slim.log. Fit separately per polarity/platform, and validate that predicted calibrant CCS values agree with the reference tune-mix CCS values within the calibrant-matching ppm tolerance (--ppm 150 in the SLIM demo).

## Related tools

- **AutoCCS** (Implements the linearized power calibration (--calib_method power) for traveling wave-based IMS; invoked as the autoCCS.py command-line entry point) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python 3.7** (Runtime environment for installing dependencies via pip/conda and executing autoCCS.py) — https://www.python.org/downloads/

## Examples

```
python autoCCS.py --calib_method power --calibrant_file TuneMix-CCS.txt --tunemix_sample_type AgilentTuneMix --sample_meta Datasets.csv --output_dir Results/ &> LogFiles/slim.log
```

## Evaluation signals

- calibration_coefficients.csv exists and contains a coefficient field for every fitted parameter of the power function: exponent, pre-exponential coefficient, and time offset
- Predicted calibrant CCS values agree with the reference tune-mix CCS values within the calibrant-matching ppm tolerance (--ppm 150 in the SLIM demo)
- Separate coefficient sets are written for each ionization polarity and platform
- Log file (LogFiles/slim.log) confirms the run completed with --calib_method power

## Limitations

- Requires reference calibrants with known CCS values (e.g. Agilent tune-mix TuneMix-CCS); cannot determine absolute CCS for TWIMS without them
- Platform-specific arrival-time corrections (tacc subtraction for SLIM, EDC delay coefficient for Waters Synapt) must be applied before fitting, or calibration will be biased
- Discussion-section content is absent from the extracted article, so no documented limitations or reproducibility gaps could be extracted

## Evidence

- [methods] AutoCCS handles TWIMS calibration by performing either polynomial or power (linearized power function) regression analysis, fitting the arrival time tA and known CCS values of reference compounds: [evidence span withheld — anchor status: not_found]
- [methods] Linearized power function [(Ruotolo,B.T. et al., 2008)]: "Linearized power function [(Ruotolo,B.T. et al., 2008)]"
- [abstract] linear calibration for Bruker timsTOF and non-linear calibration for traveling wave based-IMS in Waters Synapt and SLIM: [evidence span withheld — anchor status: not_found]
- [readme] Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
