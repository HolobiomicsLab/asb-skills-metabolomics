---
name: ion-mobility-calibration-curve-fitting
description: Use when converting ion arrival times from single-field RapidFire-DTIMS-MS,
  SLIM-based IMS, Waters Synapt TWIMS, or Bruker timsTOF into CCS values using a calibrant
  such as Agilent tune-mix; not needed for stepped-field DTIMS.
when_to_use_negative:
- Data are stepped-field DTIMS-MS measurements — AutoCCS handles these with the first-principles
  multi-field mode (--mode multi, --threshold_n_fields 5) and no calibration curve
  is fitted
- Inputs are already calibrated CCS values (e.g. vendor-reported values from Agilent
  IM-MS Browser) — this skill only converts arrival times to CCS, it does not compare
  or re-derive existing CCS determinations
- No calibrant with known reference CCS values was acquired for the run — single-field
  and traveling-wave calibration require a calibrant file; the method cannot proceed
  from feature files alone
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AutoCCS
  role: Fits the calibration curve from the calibrant CCS file and applies it to predict
    CCS for target ions in single-field and traveling-wave IMS-MS runs
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python 3.7
  role: Runtime interpreter required to execute autoCCS.py and its dependencies
  repo: https://www.python.org/downloads/
- name: conda
  role: Recommended way to create the autoccs environment from environment.yml before
    running calibration
  repo: https://docs.anaconda.com/anaconda/install/
- name: pip
  role: Installs Python dependencies via requirements.txt as the alternative to conda
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:da746d81e6b9a1ee32b17591237ce44ef4873f46dad0a7f6e7351db7cf5887d9
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ion-mobility-calibration-curve-fitting

## Summary

Fits calibration curves (linear, polynomial, or linearized power) relating measured arrival times to known calibrant CCS values so that CCS can be determined for unknown ions in single-field DTIMS and traveling-wave IMS-MS data, as automated in AutoCCS.

## When to use

Use when converting ion arrival times from single-field RapidFire-DTIMS-MS, SLIM-based IMS, Waters Synapt TWIMS, or Bruker timsTOF into CCS values using a calibrant such as Agilent tune-mix; not needed for stepped-field DTIMS.

## When NOT to use

- Data are stepped-field DTIMS-MS measurements — AutoCCS handles these with the first-principles multi-field mode (--mode multi, --threshold_n_fields 5) and no calibration curve is fitted
- Inputs are already calibrated CCS values (e.g. vendor-reported values from Agilent IM-MS Browser) — this skill only converts arrival times to CCS, it does not compare or re-derive existing CCS determinations
- No calibrant with known reference CCS values was acquired for the run — single-field and traveling-wave calibration require a calibrant file; the method cannot proceed from feature files alone

## Inputs

- IMS-MS frame metadata files (*.txt from ImsMetadata directories, e.g. from MassIVE MSV000085979)
- Feature files (*.csv from Features_csv directories)
- Calibrant CCS file (TuneMix-CCS.txt / TuneMix-CCS_POS.txt, Agilent tune-mix reference CCS values)
- autoCCS_config.xml configuration file
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns

## Outputs

- Per-ion CCS values for unknown analytes (e.g. ccs_values_table.csv)
- Fitted calibration curve relating arrival time to CCS for the run
- Log files capturing the calibration run

## How to apply

Run AutoCCS in single mode (--mode single --single_mode batch), supplying frame metadata and feature files, a calibrant CCS file (e.g. TuneMix-CCS.txt or TuneMix-CCS_POS.txt for AgilentTuneMix), and sample metadata columns for sample type, filename, and ionization polarity (--colname_for_sample_type, --colname_for_filename, --colname_for_ionization). Choose the calibration function to match the platform: linear (--degree 1) for single-field RapidFire-DTIMS-MS and Bruker timsTOF TIMS, quadratic polynomial (--degree 2 --calib_method poly) for SLIM-based IMS with --ppm 150, and for Waters Synapt TWIMS either the polynomial or the linearized power function (--calib_method power). Higher-order polynomials (cubic --degree 3, quartic --degree 4) are supported, but the degree should reflect the non-linearity of the traveling-wave response rather than being raised arbitrarily. The fitted curve maps calibrant arrival times to reference CCS, then predicts CCS for every target ion; AutoCCS reports these automatically without manual processing.

## Related tools

- **AutoCCS** (Fits the calibration curve from the calibrant CCS file and applies it to predict CCS for target ions in single-field and traveling-wave IMS-MS runs) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python 3.7** (Runtime interpreter required to execute autoCCS.py and its dependencies) — https://www.python.org/downloads/
- **conda** (Recommended way to create the autoccs environment from environment.yml before running calibration) — https://docs.anaconda.com/anaconda/install/
- **pip** (Installs Python dependencies via requirements.txt as the alternative to conda)

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- The calibration degree/method matches the platform: --degree 1 for single-field RapidFire-DTIMS-MS and Bruker timsTOF, --degree 2 --calib_method poly for SLIM, --calib_method power for Waters Synapt TWIMS
- Calibrant ion matching respects the --ppm tolerance (150 ppm used for the SLIM demo) and the calibrant sample type is declared as AgilentTuneMix
- Predicted CCS values for tune-mix ions show acceptable relative differences versus manually determined Agilent IM-MS Browser values (Table 1 of the paper)
- CCS determinations reproduce across the five IMS systems for the same ions
- The run completes and writes results to the --output_dir with a log in the LogFiles directory (e.g. slim.log, single.log)

## Limitations

- Calibration functions are only defined for single-field and traveling wave-based methods; stepped-field DTIMS CCS calculation does not use a calibration curve
- Non-linear traveling-wave behavior requires either polynomial or linearized-power fitting, so a plain linear fit may be inappropriate for TWIMS platforms
- The article's discussion section content is absent from the available context, so further caveats on calibration error budgets could not be extracted

## Evidence

- [methods] Calibration with polynomial functions: "Polynomial functions (e.g. quadratic or cubic functions)"
- [methods] Calibration with linearized power function: [evidence span withheld — anchor status: not_found]
- [readme] Polynomial degree options: "Users are allowed to apply high-order polynomial functions: quadratic (`--degree 2`), cubic (`--degree 3`), quartic (`--degree 4`), and so on."
- [readme] Linearized power option: "Also, it allows users to apply non-linear regression based on the linearized power function."
- [readme] Scope of calibration functions: "Calibration functions for single field and traveling wave-based methods"
