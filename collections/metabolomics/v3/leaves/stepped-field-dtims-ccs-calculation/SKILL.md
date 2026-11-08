---
name: stepped-field-dtims-ccs-calculation
description: Use when computing CCS values from stepped-field DTIMS-MS acquisitions
  (e.g., Agilent 6560 tune-mix infusion data) where ions are measured across multiple
  electric fields and per-field arrival times and frame metadata are available.
when_to_use_negative:
- Data are single-field DTIMS or RapidFire-DTIMS acquisitions — use AutoCCS --mode
  single (batch) with a calibrant file (e.g., TuneMix-CCS.txt, --degree 1) instead
  of stepped-field regression.
- Data come from traveling-wave platforms (Waters Synapt, SLIM-based IMS) or Bruker
  timsTOF TIMS — these require TWIMS/TIMS calibration modes (--calib_method poly/power,
  --calibrant_file) rather than the stepped-field multi-field mode.
- Ions are observed in fewer than the required number of electric fields (below --threshold_n_fields
  5) — stepped-field fitting cannot be applied to such targets.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: 'Open-source Python software that performs the stepped-field DTIMS CCS calculation:
    per-field arrival-time vs P/V regression, drift-time derivation, and CCS computation
    per ion/adduct.'
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python 3.7
  role: Runtime environment for executing autoCCS.py and its dependencies.
- name: conda
  role: Recommended environment manager for installing AutoCCS dependencies from environment.yml.
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:58930b40af289218cc7a4de2044339b21552d01b2f30ceff73a04301ebf5d68f
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# stepped-field-dtims-ccs-calculation

## Summary

Automated determination of collision cross-section (CCS) values from stepped-field drift-tube IMS-MS data by linear regression of arrival time versus electric field (P/V) across fields, following the Stow et al. 2017 method, implemented in AutoCCS.

## When to use

Use when computing CCS values from stepped-field DTIMS-MS acquisitions (e.g., Agilent 6560 tune-mix infusion data) where ions are measured across multiple electric fields and per-field arrival times and frame metadata are available.

## When NOT to use

- Data are single-field DTIMS or RapidFire-DTIMS acquisitions — use AutoCCS --mode single (batch) with a calibrant file (e.g., TuneMix-CCS.txt, --degree 1) instead of stepped-field regression.
- Data come from traveling-wave platforms (Waters Synapt, SLIM-based IMS) or Bruker timsTOF TIMS — these require TWIMS/TIMS calibration modes (--calib_method poly/power, --calibrant_file) rather than the stepped-field multi-field mode.
- Ions are observed in fewer than the required number of electric fields (below --threshold_n_fields 5) — stepped-field fitting cannot be applied to such targets.

## Inputs

- TargetList.csv of target ions/adducts (m/z and identifiers)
- autoCCS_config.xml configuration file
- ImsMetadata frame-metadata text files (*.txt, one per acquisition frame set)
- Features_csv per-field feature tables (*.csv) from the Agilent 6560 stepped-field DTIMS-MS runs

## Outputs

- Per-ion CCS results table with regression fit statistics, written to the output directory (e.g., SteppedField-DTIMS/Results/)
- Run log (e.g., LogFiles/multi.log) recording execution and excluded targets

## How to apply

Retrieve the stepped-field DTIMS inputs from the MassIVE deposit MSV000085979: TargetList.csv (target ions/adducts), autoCCS_config.xml, ImsMetadata frame-metadata text files, and Features_csv feature tables. Install AutoCCS (github.com/PNNL-Comp-Mass-Spec/AutoCCS) into a Python 3.7+ environment via conda (conda env create -f environment.yml; conda activate autoccs) or pip (pip install -r requirements.txt). Execute autoCCS.py in --mode multi, supplying the target list, config XML, frame-metadata and feature file globs, an --output_dir, and --threshold_n_fields 5 so only ions observed across at least five electric fields are processed; ions below this threshold are excluded. AutoCCS fits arrival time versus P/V per ion (Stow et al. 2017 stepped-field method) to derive drift times and compute CCS for each ion/adduct without manual processing. Results are written to the output directory (e.g., data/SteppedField-DTIMS/Results/) with run output captured in a log file such as LogFiles/multi.log.

## Related tools

- **AutoCCS** (Open-source Python software that performs the stepped-field DTIMS CCS calculation: per-field arrival-time vs P/V regression, drift-time derivation, and CCS computation per ion/adduct.) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python 3.7** (Runtime environment for executing autoCCS.py and its dependencies.)
- **conda** (Recommended environment manager for installing AutoCCS dependencies from environment.yml.)

## Examples

```
python -u autoCCS.py --target_list_file data/SteppedField-DTIMS/TargetList.csv --config_file data/SteppedField-DTIMS/autoCCS_config.xml --framemeta_files "data/SteppedField-DTIMS/ImsMetadata/*.txt" --feature_files "data/SteppedField-DTIMS/Features_csv/*.csv" --output_dir data/SteppedField-DTIMS/Results/ --threshold_n_fields 5 --mode multi
```

## Evaluation signals

- Run log (multi.log) records successful completion with no fatal errors.
- Results table reports a computed CCS with regression fit statistics for each target ion meeting the --threshold_n_fields 5 criterion.
- Ions observed in fewer than five electric fields are excluded from results.
- Computed CCS values agree with manually determined vendor-software values (e.g., Agilent IM-MS Browser) with differences under ~0.2% for most ions.
- CCS values reproduce consistently when re-run on the same MSV000085979 inputs.

## Limitations

- Targets measured across fewer than the minimum number of fields (--threshold_n_fields 5) cannot be fitted and are excluded.
- Demo/discussion coverage is limited: the deposited documentation notes no changelog/discussion content, so limitations and reproducibility gaps must be validated empirically.
- Stepped-field mode applies only to drift-tube IMS-MS; traveling-wave (SLIM, Waters Synapt) and TIMS (Bruker timsTOF) platforms require different calibration-based modes of AutoCCS.

## Evidence

- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS): [evidence span withheld — anchor status: not_found]
- [methods] threshold_n_fields flag documented in methods: "--threshold_n_fields 5"
- [readme] README stepped-field invocation: "--target_list_file data/SteppedField-DTIMS/TargetList.csv --config_file data/SteppedField-DTIMS/autoCCS_config.xml --framemeta_files "data/SteppedField-DTIMS/ImsMetadata/*.txt""
- [readme] README dataset availability: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
- [readme] README install instructions: "Install Python 3.7 (or newer) [[link]](https://www.python.org/downloads/) and use `pip` as follows to install dependencies."
