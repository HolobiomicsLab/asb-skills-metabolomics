---
name: twims-nonlinear-calibration
description: Use when CCS values must be computed from traveling wave-based IMS-MS
  data (Waters Synapt or SLIM) where arrival times require non-linear (quadratic/cubic
  polynomial or linearized power) calibration against Agilent tune-mix calibrants.
when_to_use_negative:
- Data come from drift tube IMS (stepped-field or single-field DTIMS, Bruker timsTOF),
  where a linear calibration (--degree 1) is appropriate instead of the non-linear
  TWIMS treatment
- No calibrant ions (e.g. Agilent tune-mix) with reference CCS values are available
  in the acquisition, making non-linear calibration against reference CCS impossible
- The goal is direct first-principles CCS from stepped-field DTIMS Mason-Schamp analysis
  rather than calibrant-based traveling-wave conversion
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Performs the non-linear (polynomial or linearized power) CCS calibration for
    traveling wave-based IMS-MS data in single-field batch mode
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python 3.7
  role: Runtime for executing the autoCCS.py CLI with TWIMS calibration parameters
  repo: https://www.python.org/downloads/
- name: conda
  role: Creates the autoccs environment from environment.yml to run the calibration
    workflow
  repo: https://docs.anaconda.com/anaconda/install/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:d4f8479ecc2ab340c0cd9195eb8590bacf8b28492463d2ccd138cd96d83703f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - Python 3.7
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# twims-nonlinear-calibration

## Summary

Applies AutoCCS non-linear calibration functions (polynomial or linearized power) to convert traveling-wave IMS (TWIMS/SLIM) arrival times into collision cross sections, where the nonlinear relationship between drift time and CCS precludes the linear single-field DTIMS approach.

## When to use

Use when CCS values must be computed from traveling wave-based IMS-MS data (Waters Synapt or SLIM) where arrival times require non-linear (quadratic/cubic polynomial or linearized power) calibration against Agilent tune-mix calibrants.

## When NOT to use

- Data come from drift tube IMS (stepped-field or single-field DTIMS, Bruker timsTOF), where a linear calibration (--degree 1) is appropriate instead of the non-linear TWIMS treatment
- No calibrant ions (e.g. Agilent tune-mix) with reference CCS values are available in the acquisition, making non-linear calibration against reference CCS impossible
- The goal is direct first-principles CCS from stepped-field DTIMS Mason-Schamp analysis rather than calibrant-based traveling-wave conversion

## Inputs

- Feature files (Features_csv/*.csv) from TWIMS/SLIM IMS-MS acquisitions (e.g. Waters Synapt or SLIM platform)
- Calibrant CCS reference file for Agilent tune-mix ions (e.g. TuneMix-CCS_POS.txt)
- Sample metadata table (Datasets.csv) with columns for sample type, raw file name, and ionization polarity
- AutoCCS XML config file (autoCCS_config.xml)

## Outputs

- Per-ion calibrated CCS values for TWIMS/SLIM analytes, written to the AutoCCS results directory (e.g. ccs_values_table.csv aggregating across platforms)
- Calibration model fit parameters (polynomial or linearized power) relating arrival time to CCS

## How to apply

Run AutoCCS in --mode single with --single_mode batch, supplying a calibrant file (e.g. TuneMix-CCS_POS.txt) of Agilent tune-mix reference CCS values and sample metadata (Datasets.csv with SampleType, RawFileName, IonPolarity columns). For SLIM-based IMS-MS, choose polynomial calibration via --calib_method poly with --degree 2 (quadratic) and an m/z tolerance of --ppm 150; higher orders (degree 3 cubic, degree 4 quartic) are permitted. For Waters Synapt TWIMS, use the linearized power function via --calib_method power (Ruotolo et al. 2008), which linearizes the traveling-wave relationship. The rationale is that traveling-wave drift times do not map linearly to CCS, unlike stepped- or single-field DTIMS where a linear fit (--degree 1) suffices, so a non-linear fit against calibrant ions is required for accurate CCS determination.

## Related tools

- **AutoCCS** (Performs the non-linear (polynomial or linearized power) CCS calibration for traveling wave-based IMS-MS data in single-field batch mode) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python 3.7** (Runtime for executing the autoCCS.py CLI with TWIMS calibration parameters) — https://www.python.org/downloads/
- **conda** (Creates the autoccs environment from environment.yml to run the calibration workflow) — https://docs.anaconda.com/anaconda/install/

## Examples

```
python -u autoCCS.py --config_file data/SLIM-IMS/autoCCS_config.xml --feature_files "data/SLIM-IMS/Features_csv/*.csv" --calibrant_file data/SLIM-IMS/TuneMix-CCS_POS.txt --mode single --single_mode batch --degree 2 --calib_method poly --ppm 150
```

## Evaluation signals

- Calibrated CCS values for Agilent tune-mix ions fall within acceptable relative differences of manually determined vendor (Agilent IM-MS Browser) values reported in Table 1 / Section 3.1
- CCS determinations show excellent reproducibility of per-ion CCS values across the five IMS systems, including the TWIMS/SLIM platforms
- The --ppm 150 m/z tolerance correctly matches calibrant and analyte features; matching calibrant ions should be recovered for all tune-mix entries in TuneMix-CCS_POS.txt
- Choosing --degree 2 (quadratic) over --degree 1 reduces calibration residuals for TWIMS arrival times, consistent with the non-linear drift-time/CCS relationship
- Output CSV contains per-ion CCS values and relative differences versus vendor values without manual per-file processing

## Limitations

- Non-linear calibration requires reference calibrant ions with known CCS (Agilent tune-mix); it cannot produce absolute CCS without them
- The linearized power function option (--calib_method power) is specific to traveling wave-based methods and follows Ruotolo et al. 2008; applying it to DTIMS data is inappropriate
- The article's discussion section is absent from the indexed context, so limitations and reproducibility gaps cannot be enumerated further

## Evidence

- [abstract] AutoCCS supports ... linear calibration for Bruker timsTOF and non-linear calibration for traveling wave based-IMS in Waters Synapt and SLIM.: [evidence span withheld — anchor status: not_found]
- [readme] SLIM command with degree 2 poly and ppm 150: "--single_mode batch --degree 2 --calib_method poly --ppm 150 &> data/SLIM-IMS/LogFiles/slim.log"
- [readme] README on power function: "Also, it allows users to apply non-linear regression based on the linearized power function.
```bash
  --calib_method power
```"
- [methods] Polynomial functions calibration: [evidence span withheld — anchor status: not_found]
- [readme] README on high-order polynomials: "Users are allowed to apply high-order polynomial functions: quadratic (`--degree 2`), cubic (`--degree 3`), quartic (`--degree 4`), and so on."
