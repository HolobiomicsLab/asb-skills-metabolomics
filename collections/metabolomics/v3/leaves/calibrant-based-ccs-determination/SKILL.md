---
name: calibrant-based-ccs-determination
description: Use when you have single-field DTIMS-MS feature files plus IMS frame
  metadata (e.g., RapidFire SPE-IMS-MS runs) and need calibrated CCS values for analytes
  such as rotenone using Agilent tune-mix calibrant CCS.
when_to_use_negative:
- Data were acquired in stepped-field DTIMS mode with multiple fields per analyte
  — use AutoCCS --mode multi instead of calibrant-based single-field
- Data come from traveling wave-based IMS (TWIMS/SLIM or Bruker timsTOF), which require
  traveling-wave or linearized power calibration functions rather than the DTIMS single-field
  method
- Input is already a table of calibrated CCS values with no raw IMS feature/frame
  metadata available
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Open-source Python software that performs the single-field calibrant-based
    CCS calculation, including batch mode calibration with Agilent tune-mix and the
    enhanced pressure/temperature-corrected variant
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Runtime for executing autoCCS.py and parsing AutoCCS result outputs to compute
    per-run CCS statistics (mean, %RSD)
- name: conda
  role: Environment manager used to create the autoccs environment from environment.yml
    before running the pipeline
- name: pip
  role: Alternative installer for AutoCCS dependencies via requirements.txt when conda
    is not used
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:79bb5383338b56986a3b525402ac7ac007d18730d1e806d6f9589d17e2057bcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# calibrant-based-ccs-determination

## Summary

Determines collision cross-section (CCS) values from single-field DTIMS-MS measurements by calibrating against reference calibrant ions (e.g., Agilent tune-mix) using AutoCCS, optionally with per-run pressure and temperature corrections to stabilize CCS across instrument condition fluctuations.

## When to use

Use when you have single-field DTIMS-MS feature files plus IMS frame metadata (e.g., RapidFire SPE-IMS-MS runs) and need calibrated CCS values for analytes such as rotenone using Agilent tune-mix calibrant CCS.

## When NOT to use

- Data were acquired in stepped-field DTIMS mode with multiple fields per analyte — use AutoCCS --mode multi instead of calibrant-based single-field
- Data come from traveling wave-based IMS (TWIMS/SLIM or Bruker timsTOF), which require traveling-wave or linearized power calibration functions rather than the DTIMS single-field method
- Input is already a table of calibrated CCS values with no raw IMS feature/frame metadata available

## Inputs

- IMS-MS feature files (CSV) from single-field RapidFire-DTIMS-MS runs
- IMS frame-metadata files (txt) with per-run pressure and temperature
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns
- Calibrant CCS list (TuneMix-CCS.txt) with Agilent tune-mix reference CCS values
- AutoCCS XML config file (autoCCS_config.xml)

## Outputs

- Per-run calibrated CCS values for target analytes (e.g., protonated rotenone) for each calibration variant
- Comparison table of mean CCS and %RSD across runs for conventional vs. enhanced single-field calibration
- AutoCCS result files and log files

## How to apply

Download the SPE-IMS-MS feature files, IMS frame-metadata files, sample metadata table, and calibrant CCS list from MassIVE deposit MSV000085979. Run AutoCCS in single-field batch mode with the conventional single-field method (Kurulugama et al., 2015), providing --tunemix_sample_type AgilentTuneMix, the TuneMix-CCS.txt calibrant file, the autoCCS_config.xml, and a linear calibration function (--degree 1), mapping sample type, raw filename, and ionization polarity columns from the sample metadata. For datasets where per-run instrument condition fluctuations inflate CCS variance, re-run with the enhanced single-field calibration that incorporates per-run pressure and temperature corrections into the CCS calculation. Parse both AutoCCS result outputs to extract per-run analyte CCS values and compare mean CCS and %RSD between calibration variants.

## Related tools

- **AutoCCS** (Open-source Python software that performs the single-field calibrant-based CCS calculation, including batch mode calibration with Agilent tune-mix and the enhanced pressure/temperature-corrected variant) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Runtime for executing autoCCS.py and parsing AutoCCS result outputs to compute per-run CCS statistics (mean, %RSD))
- **conda** (Environment manager used to create the autoccs environment from environment.yml before running the pipeline)
- **pip** (Alternative installer for AutoCCS dependencies via requirements.txt when conda is not used)

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt" --feature_files "data/SingleField-RapidFire-DTIMS/Features_csv/*.csv" --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --output_dir data/SingleField-RapidFire-DTIMS/Results/ --tunemix_sample_type AgilentTuneMix --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- Enhanced-calibration rotenone CCS distribution should match the reported mean of 194.7 with 0.4% RSD, versus conventional calibration at 195.6 with 1.4% RSD
- Enhanced single-field calibration should yield an almost single CCS distribution across the 7106 RapidFire runs, indicating per-run pressure/temperature correction removed condition-driven variance
- AutoCCS run should complete with a log file written and result outputs generated in --output_dir for both calibration variants
- Calibration should use a linear function (--degree 1) fitted against the Agilent tune-mix calibrant CCS values

## Limitations

- Single-field calibrant-based CCS is sensitive to per-run instrument condition fluctuations (pressure and temperature), which the conventional method leaves as inflated variance; the enhanced variant mitigates but does not eliminate this
- Single-field calibrant CCS determination requires per-run frame metadata including pressure and temperature; without these, only the conventional calibration is possible
- The method is specific to drift tube-based IMS (DTIMS); traveling wave platforms need different calibration functions (polynomial or linearized power)

## Evidence

- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS), single-field methods for DTIMS (standard and enhanced calibration), linear calibration for Bruker timsTOF and non-linear calibration for traveling wave based-IMS: [evidence span withheld — anchor status: not_found]
- [methods] Single field method [(Kurulugama,R.T. et al., 2015)]: "Single field method [(Kurulugama,R.T. et al., 2015)]"
- [readme] Demo dataset is publicly available at MassIVE (Accession: MSV000085979): "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979](https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000085979))"
- [methods] Install Python 3.7 (or newer) and use pip as follows to install dependencies.: "Install Python 3.7 (or newer) [[link]](https://www.python.org/downloads/) and use `pip` as follows to install dependencies."
