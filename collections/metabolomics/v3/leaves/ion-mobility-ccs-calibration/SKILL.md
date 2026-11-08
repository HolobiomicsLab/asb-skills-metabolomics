---
name: ion-mobility-ccs-calibration
description: Use when you have IMS-MS arrival-time/feature data (e.g. UIMF frame metadata
  plus Features_csv tables) and need to convert drift or arrival times into calibrated
  CCS values, choosing among single-field, stepped-field, or traveling-wave calibration
  methods.
when_to_use_negative:
- You only have raw MS data without ion mobility arrival-time information or frame
  metadata — CCS calibration requires IMS feature/arrival-time tables.
- You need manually curated or stepped-field multi-field CCS with fewer than the required
  number of field strengths — the stepped-field mode uses --threshold_n_fields 5 to
  gate datasets.
- Your analytes' arrival times fall outside the calibrant range — extrapolation can
  substantially inflate error (Pterin error was 2.26% when outside the TAA calibrant
  range).
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AutoCCS
  role: Open-source Python software that performs automated CCS calculation and calibration
    for DTIMS, timsTOF, TWIMS, and SLIM IMS-MS data
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: conda
  role: Creates the autoccs environment from environment.yml for installation
- name: pip
  role: Alternative installer for AutoCCS dependencies via requirements.txt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:2ad7c34110f1ae6d2c751adfd6f3eb3616f73a60d2bce700847167bb214b950a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# ion-mobility-ccs-calibration

## Summary

Automated collision cross-section (CCS) calculation and calibration from IMS-MS measurements using AutoCCS, supporting DTIMS (stepped- and single-field), Bruker timsTOF, and traveling-wave platforms (Waters Synapt, SLIM) with linear, polynomial, or linearized power calibration functions.

## When to use

Use when you have IMS-MS arrival-time/feature data (e.g. UIMF frame metadata plus Features_csv tables) and need to convert drift or arrival times into calibrated CCS values, choosing among single-field, stepped-field, or traveling-wave calibration methods.

## When NOT to use

- You only have raw MS data without ion mobility arrival-time information or frame metadata — CCS calibration requires IMS feature/arrival-time tables.
- You need manually curated or stepped-field multi-field CCS with fewer than the required number of field strengths — the stepped-field mode uses --threshold_n_fields 5 to gate datasets.
- Your analytes' arrival times fall outside the calibrant range — extrapolation can substantially inflate error (Pterin error was 2.26% when outside the TAA calibrant range).

## Inputs

- IMS-MS feature tables (Features_csv/*.csv) with one row per detected feature
- ImsMetadata frame-metadata files (e.g. UIMF-derived *.txt)
- autoCCS_config.xml configuration file
- TargetList.csv of analyte m/z targets
- Calibrant reference CCS file (e.g. TuneMix-CCS.txt / TuneMix-CCS_POS.txt with TAA salt or Agilent tune-mix reference CCS values)
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns

## Outputs

- Calibrated CCS values for calibrants and unknown/standard analytes written to a Results output directory
- Log files capturing the run (e.g. single.log, multi.log)

## How to apply

Install AutoCCS (Python 3.7+ via conda environment.yml or pip requirements.txt) from the PNNL-Comp-Mass-Spec repo. Prepare a target list and autoCCS config XML, point --framemeta_files at ImsMetadata files and --feature_files at Features_csv/*.csv. For single-field DTIMS or SLIM (traveling-wave) data, supply a calibrant reference file (e.g. TuneMix-CCS_POS.txt with TAA/tune-mix reference CCS values) and map metadata columns (--colname_for_sample_type SampleType, --colname_for_filename RawFileName, --colname_for_ionization IonPolarity). Choose the calibration function by data regime: linear (--degree 1) for single-field DTIMS, polynomial (e.g. --degree 2, --ppm 150) or linearized power (--calib_method power) for traveling-wave SLIM data; when comparing against DTIMS reference CCS, the linearized power function tends to agree better, but check whether analyte arrival times fall inside the calibrant range (e.g. Pterin fell outside the TAA calibrant range, inflating error to ~2.26%). Run in batch mode (--mode single --single_mode batch, or --mode multi with --threshold_n_fields for stepped-field) and extract fitted CCS values from the Results output directory.

## Related tools

- **AutoCCS** (Open-source Python software that performs automated CCS calculation and calibration for DTIMS, timsTOF, TWIMS, and SLIM IMS-MS data) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **conda** (Creates the autoccs environment from environment.yml for installation)
- **pip** (Alternative installer for AutoCCS dependencies via requirements.txt)

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt" --feature_files "data/SingleField-RapidFire-DTIMS/Features_csv/*.csv" --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --output_dir data/SingleField-RapidFire-DTIMS/Results/ --tunemix_sample_type AgilentTuneMix --sample_meta data/SingleField-RapidFire-DTIMS/Datasets.csv --colname_for_sample_type SampleType --colname_for_filename RawFileName --colname_for_ionization IonPolarity --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- Results output directory exists and contains a table with CCS columns for each calibration method run.
- Computed CCS for standards (e.g. Agilent tune-mix or IROA compounds Adenine, Trehalose, Pterin) agree with DTIMS reference values within the accuracy reported across platforms; relative deviations should be tabulated per calibration method.
- Calibration regression uses the expected function: --degree 1 (linear) for single-field DTIMS, --degree 2 quadratic polynomial or --calib_method power for SLIM traveling-wave data.
- Datasets are correctly assigned via SampleType/RawFileName/IonPolarity metadata mapping (tunemix_sample_type AgilentTuneMix identifies calibrant runs).
- Check whether analyte arrival times lie within the calibrant range; unusually large residuals (e.g. ~2.26% for Pterin) indicate extrapolation.

## Limitations

- Linearized power regression can give larger errors when an analyte's arrival time falls outside the calibrant range (Pterin: 2.26% error vs DTIMS reference).
- The choice between polynomial and linearized power calibration materially affects agreement with DTIMS reference CCS values; the better method must be established empirically per dataset.
- The article's discussion section content is absent, so no additional limitations or reproducibility gaps can be extracted from this document.

## Evidence

- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS), single-field methods for DTIMS (standard and enhanced calibration), linear calibration for Bruker timsTOF and non-linear calibration for traveling wave based-IMS in Waters Synapt and SLIM.: [evidence span withheld — anchor status: not_found]
- [readme] README installation instructions: "Install Python 3.7 (or newer) [[link]](https://www.python.org/downloads/) and use `pip` as follows to install dependencies."
- [readme] Calibration functions listed in README: "- Calibration functions for single field and traveling wave-based methods
  - Linear function
  - Polynomial functions (e.g. quadratic or cubic functions)"
- [methods] Demo data availability: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
- [results] Task finding on linearized power vs polynomial for IROA standards: [evidence span withheld — anchor status: not_found]
