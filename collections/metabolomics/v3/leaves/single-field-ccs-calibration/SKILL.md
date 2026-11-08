---
name: single-field-ccs-calibration
description: Use when you have single-field RapidFire-DTIMS-MS (or similar DTIMS)
  feature files plus IMS frame-metadata from many runs and need calibrated per-run
  CCS values, e.g. tracking CCS variance of a compound like protonated rotenone across
  thousands of SPE-IMS-MS plant-extract runs.
when_to_use_negative:
- Input is stepped-field DTIMS data collected at multiple drift fields — use AutoCCS
  --mode multi with --threshold_n_fields instead of the single-field calibration
- Data are traveling-wave based (Waters Synapt, SLIM) or Bruker timsTOF, where traveling-wave-based
  or non-linear calibration functions are required rather than the single-field DTIMS
  method
- Feature files or IMS frame metadata are absent, since calibration depends on pairing
  per-frame instrument conditions with extracted features
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AutoCCS
  role: 'Performs the automated CCS calculation: calibrates single-field DTIMS drift
    times with Agilent tune-mix CCS values using a linear function (--degree 1, --single_mode
    batch) and applies per-run pressure/temperature corrections in the enhanced single-field
    mode'
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Runtime for autoCCS.py; Python 3.7 or newer with pip-installed dependencies
    or a conda environment from environment.yml
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: pip
  role: Installs AutoCCS dependencies via pip install -r requirements.txt when not
    using conda
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: conda
  role: 'Recommended installation route: creates the autoccs environment from environment.yml
    (conda env create -f environment.yml; conda activate autoccs)'
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:c97a2f15bb6224720c59eb3f3764f543e71b0f79f19db992d68fb07c1f20c519
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - conda
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-field-ccs-calibration

## Summary

Compute collision cross sections (CCS) from single-field DTIMS-MS data by running AutoCCS in single-field batch mode, calibrating with Agilent tune-mix CCS values and a linear function, and optionally applying per-run pressure/temperature corrections to reduce CCS drift across instrument condition fluctuations.

## When to use

Use when you have single-field RapidFire-DTIMS-MS (or similar DTIMS) feature files plus IMS frame-metadata from many runs and need calibrated per-run CCS values, e.g. tracking CCS variance of a compound like protonated rotenone across thousands of SPE-IMS-MS plant-extract runs.

## When NOT to use

- Input is stepped-field DTIMS data collected at multiple drift fields — use AutoCCS --mode multi with --threshold_n_fields instead of the single-field calibration
- Data are traveling-wave based (Waters Synapt, SLIM) or Bruker timsTOF, where traveling-wave-based or non-linear calibration functions are required rather than the single-field DTIMS method
- Feature files or IMS frame metadata are absent, since calibration depends on pairing per-frame instrument conditions with extracted features

## Inputs

- IMS frame-metadata files (*.txt, e.g. data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt)
- Feature CSV files per dataset (data/SingleField-RapidFire-DTIMS/Features_csv/*.csv)
- Calibrant CCS list file (TuneMix-CCS.txt, Agilent tune-mix)
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns
- AutoCCS XML config file (autoCCS_config.xml)

## Outputs

- AutoCCS result outputs with per-run calibrated CCS values for each analyte
- Comparison of mean CCS and %RSD of the target ion across runs for conventional vs enhanced calibration (e.g. rotenone: 195.6 Å², 1.4% RSD conventional vs 194.7 Å², 0.4% RSD enhanced)

## How to apply

Download the feature files, IMS frame-metadata files, sample metadata table, and calibrant CCS list (MassIVE MSV000085979 for the demo). Install AutoCCS via conda (environment.yml) or pip (requirements.txt, Python 3.7+). Run AutoCCS in single-field batch mode: point --framemeta_files and --feature_files at the frame metadata and feature CSVs, supply the Agilent tune-mix calibrant CCS file with --tunemix_sample_type AgilentTuneMix, set --calibrant_file TuneMix-CCS.txt, use the linear calibration function with --degree 1, and pass --sample_meta with the sample-type and filename column names (--colname_for_sample_type, --colname_for_filename, --colname_for_ionization). First run the conventional single-field method (Kurulugama et al., 2015), then re-run with the enhanced single-field calibration that incorporates per-run pressure and temperature into the CCS calculation. Finally parse the AutoCCS result outputs and compute the mean CCS and %RSD of the target ion (e.g. protonated rotenone) across runs to compare calibration variants.

## Related tools

- **AutoCCS** (Performs the automated CCS calculation: calibrates single-field DTIMS drift times with Agilent tune-mix CCS values using a linear function (--degree 1, --single_mode batch) and applies per-run pressure/temperature corrections in the enhanced single-field mode) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Runtime for autoCCS.py; Python 3.7 or newer with pip-installed dependencies or a conda environment from environment.yml) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **pip** (Installs AutoCCS dependencies via pip install -r requirements.txt when not using conda) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **conda** (Recommended installation route: creates the autoccs environment from environment.yml (conda env create -f environment.yml; conda activate autoccs)) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt" --feature_files "data/SingleField-RapidFire-DTIMS/Features_csv/*.csv" --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --output_dir data/SingleField-RapidFire-DTIMS/Results/ --tunemix_sample_type AgilentTuneMix --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- Enhanced single-field calibration of protonated rotenone across the RapidFire SPE-IMS-MS runs should yield mean CCS ≈ 194.7 with ≈ 0.4% RSD, matching Figure 3 of the paper
- Conventional single-field calibration (Kurulugama et al., 2015) on the same runs should reproduce mean CCS ≈ 195.6 with ≈ 1.4% RSD, providing a negative-control baseline
- The enhanced-calibration CCS distribution should be almost a single distribution, not split into multiple condition-dependent modes
- Log output (e.g. data/SingleField-RapidFire-DTIMS/LogFiles/single.log) should complete without errors and the --output_dir Results directory should contain per-dataset CCS result files
- Calibration is restricted to a linear function (--degree 1) for the single-field mode; verify the degree parameter was honored in the run log

## Limitations

- Single-field calibration is specific to drift tube (DTIMS) measurements; traveling-wave platforms need the TWIMS method and possibly non-linear calibration functions
- CCS accuracy degrades with instrument condition fluctuations unless the enhanced per-run pressure and temperature correction is applied (conventional method showed 1.4% RSD vs 0.4% enhanced)
- Discussion-section content is absent from the source document, so limitations and reproducibility gaps beyond those stated in abstract/methods cannot be extracted

## Evidence

- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS), single-field methods for DTIMS (standard and enhanced calibration), linear calibration for Bruker timsTOF: [evidence span withheld — anchor status: not_found]
- [abstract] We present AutoCCS, an open-source software to rapidly determine CCS values from IMS-MS measurements.: "We present AutoCCS, an open-source software to rapidly determine CCS values from IMS-MS measurements."
- [readme] Demo dataset is publicly available at MassIVE (Accession: MSV000085979): "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
- [readme] Calibration functions for single field and traveling wave-based methods - Linear function: "Calibration functions for single field and traveling wave-based methods
  - Linear function"
