---
name: single-field-dtims-arrival-time-calibration
description: Use when calibrating single-field DTIMS arrival/drift times to CCS via
  external calibrants (Agilent tune-mix with known CCS values), especially when calibrant
  and sample runs differ in pressure/temperature.
when_to_use_negative:
- Stepped-field DTIMS data — use AutoCCS's stepped-field method (--threshold_n_fields
  5, --mode multi) instead, which determines CCS directly without a calibration curve
- Traveling wave-based IMS (TWIMS/SLIM) data — these require polynomial or linearized
  power calibration functions rather than the single-field linear (degree 1) function
- Feature tables without an accompanying calibrant CCS reference file (e.g., TuneMix-CCS.txt),
  since external calibration depends on known calibrant CCS values
tools:
- name: AutoCCS
  role: 'Runs the single-field calibration: computes reduced and P/T-adjusted calibrant
    CCS, fits the linear calibration function (--degree 1) and applies it to sample
    ions'
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Runtime for autoCCS.py; environment installed via conda (environment.yml)
    or pip (requirements.txt) with Python 3.7 or newer
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:8a767e32665b5a21a96dc58d841d943d136a2be531b8e0458f45c758de24a6a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-field-dtims-arrival-time-calibration

## Summary

Builds a linear arrival-time-to-CCS calibration curve for single-field DTIMS-MS using known calibrant ions (e.g., Agilent tune-mix), correcting the measured drift time for charge and reduced mass and applying a pressure/temperature (P/T) adjustment before least-squares fitting.

## When to use

Use when calibrating single-field DTIMS arrival/drift times to CCS via external calibrants (Agilent tune-mix with known CCS values), especially when calibrant and sample runs differ in pressure/temperature.

## When NOT to use

- Stepped-field DTIMS data — use AutoCCS's stepped-field method (--threshold_n_fields 5, --mode multi) instead, which determines CCS directly without a calibration curve
- Traveling wave-based IMS (TWIMS/SLIM) data — these require polynomial or linearized power calibration functions rather than the single-field linear (degree 1) function
- Feature tables without an accompanying calibrant CCS reference file (e.g., TuneMix-CCS.txt), since external calibration depends on known calibrant CCS values

## Inputs

- Calibrant feature tables (Features_csv/*.csv) from single-field RapidFire-DTIMS-MS runs
- Reference calibrant CCS list (TuneMix-CCS.txt with known Agilent tune-mix CCS values)
- autoCCS_config.xml configuration file
- ImsMetadata frame metadata files (*.txt)
- Sample metadata CSV (Datasets.csv) with columns for SampleType, RawFileName and IonPolarity

## Outputs

- Fitted linear calibration curve (slope, intercept, R^2) relating P/T-adjusted reduced CCS (Omega'') to corrected arrival time
- CCS values assigned to calibrant and unknown analyte ions in the results directory

## How to apply

Load the calibrant feature tables from the single-field RapidFire-DTIMS Features_csv directory together with the reference CCS list TuneMix-CCS.txt (demo data from MassIVE MSV000085979). For each calibrant ion, compute the reduced CCS Omega' = (1/z)·sqrt(mI/(mI+mB))·Omega using the ion charge state z, ion mass mI and buffer gas mass mB (Eq. 3), then apply the P/T adjustment Omega'' = (P/T)·Omega' to account for pressure/temperature differences between calibrant and sample runs. Fit the linear calibration function (Eq. 4) of Omega'' versus corrected arrival time by least-squares regression at degree 1 (matching AutoCCS's --degree 1 setting for single-field DTIMS), and record the fitted slope, intercept and coefficient of determination (R^2). The fitted line is then used to assign CCS to unknown analyte ions in sample feature tables via the single_mode batch workflow.

## Related tools

- **AutoCCS** (Runs the single-field calibration: computes reduced and P/T-adjusted calibrant CCS, fits the linear calibration function (--degree 1) and applies it to sample ions) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Runtime for autoCCS.py; environment installed via conda (environment.yml) or pip (requirements.txt) with Python 3.7 or newer)

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt" --feature_files "data/SingleField-RapidFire-DTIMS/Features_csv/*.csv" --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --output_dir data/SingleField-RapidFire-DTIMS/Results/ --tunemix_sample_type AgilentTuneMix --sample_meta data/SingleField-RapidFire-DTIMS/Datasets.csv --colname_for_sample_type SampleType --colname_for_filename RawFileName --colname_for_ionization IonPolarity --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- Least-squares fit converges and R^2 is reported alongside slope and intercept (the paper reports no numeric R^2 threshold, so a qualitative high-fit check applies)
- Calibration is fit at degree 1 (linear), matching the documented --degree 1 setting for single-field DTIMS
- Calibrated tune-mix CCS values agree with the reference values in TuneMix-CCS.txt
- Output files are written to the specified --output_dir with sample CCS results derived via --single_mode batch
- Each calibrant ion's Omega'' incorporates its charge state and reduced mass correction per Eq. 3 before fitting

## Limitations

- The paper reports no numeric R^2 threshold, so calibration quality is judged qualitatively by a high-fit check
- Discussion section content is absent; no limitations, missing artifacts, or reproducibility gaps could be extracted from the document
- Single-field external calibration depends on known calibrant CCS values and correct ion charge states; it is not the direct CCS determination available from the stepped-field method

## Evidence

- [methods] Calibration functions for single field and traveling wave-based methods
  - Linear function: "Calibration functions for single field and traveling wave-based methods
  - Linear function"
- [methods] single-field RapidFire-DTIMS-MS, SLIM-based IMS: "single-field RapidFire-DTIMS-MS, SLIM-based IMS"
- [readme] python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files ... --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt ... --degree 1 --single_mode batch --mode single: "python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml"
- [readme] Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
