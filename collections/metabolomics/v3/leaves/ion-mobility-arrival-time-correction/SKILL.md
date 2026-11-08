---
name: ion-mobility-arrival-time-correction
description: Use when working with traveling-wave IMS-MS data (SLIM-based IMS or Waters
  Synapt) where arrival times from feature files must be adjusted for instrument-specific
  delays before fitting CCS calibration functions.
when_to_use_negative:
- Data are DTIMS-MS (stepped-field or single-field RapidFire), where CCS is computed
  from the fundamental low-field ion mobility equation rather than by arrival-time-based
  empirical calibration
- Input features already carry corrected arrival times or calibrated CCS values, so
  applying a second tacc/EDC adjustment would double-correct
- The instrument is Bruker timsTOF, which uses a linear TIMS calibration rather than
  the traveling-wave polynomial/power correction described here
tools:
- name: AutoCCS
  role: Applies the platform-specific arrival-time adjustment and fits the TWIMS polynomial
    or linearized power calibration function via autoCCS.py CLI flags (--calib_method,
    --degree, --ppm)
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: UIMF-Library
  role: C# library for reading UIMF frame metadata, including the TOFCorrectionTime
    time-delay correction used in arrival-time adjustment
  repo: PNNL-Comp-Mass-Spec/UIMF-Library
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:425b405ece11c9023936af4290de11942e5efa1a632e1e70f0d06b1fca9cb0f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - UIMF-Library
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# ion-mobility-arrival-time-correction

## Summary

Applies platform-specific arrival-time adjustments to TWIMS-MS data — subtracting the accumulation/access time (tacc) for SLIM-based IMS and applying the EDC delay coefficient for Waters Synapt — so that corrected arrival times can be regressed against known calibrant CCS values to build TWIMS calibration functions.

## When to use

Use when working with traveling-wave IMS-MS data (SLIM-based IMS or Waters Synapt) where arrival times from feature files must be adjusted for instrument-specific delays before fitting CCS calibration functions.

## When NOT to use

- Data are DTIMS-MS (stepped-field or single-field RapidFire), where CCS is computed from the fundamental low-field ion mobility equation rather than by arrival-time-based empirical calibration
- Input features already carry corrected arrival times or calibrated CCS values, so applying a second tacc/EDC adjustment would double-correct
- The instrument is Bruker timsTOF, which uses a linear TIMS calibration rather than the traveling-wave polynomial/power correction described here

## Inputs

- TWIMS-MS feature files (CSV) with arrival times for calibrants and analytes (e.g. Agilent tune-mix from MassIVE MSV000085979)
- Calibrant arrival-time/CCS reference table (e.g. TuneMix-CCS.txt)
- Frame metadata files (e.g. UIMF-derived ImsMetadata .txt files with accumulation/access-time or TOFCorrectionTime fields)
- Sample metadata CSV mapping raw filenames, sample types, and ionization polarity

## Outputs

- calibration_coefficients.csv with fitted polynomial degree terms or power-function exponent/prefactor plus time offset, per ionization polarity and platform
- Run log (e.g. LogFiles/slim.log)
- Calibrated TWIMS CCS values for unknown analyte ions

## How to apply

Load the calibrant arrival-time/CCS reference table (e.g. Agilent tune-mix TuneMix-CCS values from MassIVE MSV000085979) and the matching feature files in the AutoCCS Python 3.7/conda 'autoccs' environment. Before any regression, apply the traveling-wave-based method's platform-specific adjustment: subtract the accumulation/access time (tacc) for SLIM-based IMS data, or apply the EDC delay coefficient for Waters Synapt data — for UIMF-format data the frame metadata exposes TOFCorrectionTime as the time-delay correction. Then fit the corrected arrival times against known CCS values by choosing either a polynomial function (quadratic or cubic, --calib_method poly with --degree 2 or 3) or the linearized power function of Ruotolo et al. 2008 (--calib_method power), which jointly estimates the exponent, pre-exponential coefficient, and time offset. Write fitted coefficients per ionization polarity and platform to calibration_coefficients.csv and log the run (e.g. LogFiles/slim.log). The rationale is that TWIMS CCS cannot be computed directly from the fundamental low-field ion mobility equation, so empirical calibration against reference compounds is required.

## Related tools

- **AutoCCS** (Applies the platform-specific arrival-time adjustment and fits the TWIMS polynomial or linearized power calibration function via autoCCS.py CLI flags (--calib_method, --degree, --ppm)) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **UIMF-Library** (C# library for reading UIMF frame metadata, including the TOFCorrectionTime time-delay correction used in arrival-time adjustment) — PNNL-Comp-Mass-Spec/UIMF-Library

## Examples

```
python autoCCS.py --config_file data/SLIM/autoCCS_config.xml --calibrant_file data/SLIM/TuneMix-CCS.txt --calib_method power --degree 3 --ppm 150 --mode single &> data/SLIM/LogFiles/slim.log
```

## Evaluation signals

- calibration_coefficients.csv exists and contains a coefficient field for every fitted parameter (polynomial degree terms, or power-function exponent and prefactor plus time offset)
- Coefficients are written for each ionization polarity and platform separately
- Predicted calibrant CCS values agree with the reference Agilent tune-mix CCS values within the calibrant-matching ppm tolerance (--ppm 150 in the SLIM demo)
- Run log (LogFiles/slim.log) is produced and records the fitting output
- For UIMF inputs, the frame metadata TOFCorrectionTime field is present and was consumed by the correction

## Limitations

- Arrival-time adjustment is platform-specific — SLIM requires subtracting tacc while Waters Synapt requires the EDC delay coefficient — so the wrong adjustment for the platform invalidates the calibration
- The article's Discussion section is absent from the source document, so limitations and reproducibility gaps cannot be fully characterized
- The README's SLIM demo command appears commented out, and the README is truncated, so exact SLIM CLI parameters beyond those documented in the workflow must be confirmed from the repository

## Evidence

- [methods] Linearized power function [(Ruotolo,B.T. et al., 2008)]: "Linearized power function [(Ruotolo,B.T. et al., 2008)]"
- [methods] Polynomial functions (e.g. quadratic or cubic functions): "Polynomial functions (e.g. quadratic or cubic functions)"
- [methods] we demonstrated the CCS determination using AutoCCS for the Agilent tune-mix samples in three different platforms: stepped-field DTIMS-MS: "we demonstrated the CCS determination using AutoCCS for the Agilent tune-mix samples in three different platforms: stepped-field DTIMS-MS"
- [readme] Traveling wave based-IMS MS (TWIMS-MS) such as Structures for Lossless Ion Manipulations (SLIM)-based IMS-MS: "Traveling wave based-IMS MS (TWIMS-MS) such as Structures for Lossless Ion Manipulations (SLIM)-based IMS-MS"
- [readme] TOFCorrectionTime | float | Time Delay correction: [evidence span withheld — anchor status: external_doc]
