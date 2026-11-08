---
name: collision-cross-section-calculation
description: Use when raw IMS-MS frame metadata and feature files must be converted
  into per-ion CCS values, e.g. Agilent tune-mix ions measured across DTIMS, RapidFire-DTIMS,
  SLIM, Waters Synapt, or Bruker timsTOF platforms.
when_to_use_negative:
- The input is already a table of computed CCS values and the task is only aggregation
  or comparison against vendor values — no re-calculation is needed
- The data were not acquired on a supported IMS platform/method (stepped-field DTIMS,
  single-field DTIMS, or traveling-wave-based IMS such as SLIM or Waters Synapt)
- No calibrant reference file (e.g. Agilent tune-mix CCS values) is available for
  single-field or TWIMS calibration
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Primary software that computes CCS values from IMS-MS measurements in multi
    (stepped-field) or single-field batch modes
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python 3.7
  role: Runtime environment in which autoCCS.py is executed
- name: conda
  role: Recommended installer that creates the autoccs environment from environment.yml
- name: pip
  role: Alternative installer of dependencies via requirements.txt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:a3e7a1e0fb72870ed2311f575d17be9aeeeab8d2f8a8dc9c112e0904786d1367
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
yes_rate: 0.6470588235294118
n_claims: 17
---

# collision-cross-section-calculation

## Summary

Automated determination of collision cross-section (CCS) values from IMS-MS measurements using AutoCCS, an open-source Python tool that supports stepped-field DTIMS, single-field DTIMS, and traveling-wave (SLIM/Synapt) methods with linear, polynomial, or linearized-power calibration functions.

## When to use

Use when raw IMS-MS frame metadata and feature files must be converted into per-ion CCS values, e.g. Agilent tune-mix ions measured across DTIMS, RapidFire-DTIMS, SLIM, Waters Synapt, or Bruker timsTOF platforms.

## When NOT to use

- The input is already a table of computed CCS values and the task is only aggregation or comparison against vendor values — no re-calculation is needed
- The data were not acquired on a supported IMS platform/method (stepped-field DTIMS, single-field DTIMS, or traveling-wave-based IMS such as SLIM or Waters Synapt)
- No calibrant reference file (e.g. Agilent tune-mix CCS values) is available for single-field or TWIMS calibration

## Inputs

- Frame metadata text files (--framemeta_files, e.g. ImsMetadata/*.txt)
- Feature CSV files (--feature_files, e.g. Features_csv/*.csv)
- Target list CSV for stepped-field mode (TargetList.csv)
- XML configuration file (autoCCS_config.xml)
- Calibrant CCS reference file (e.g. TuneMix-CCS.txt or TuneMix-CCS_POS.txt)
- Sample metadata CSV (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns

## Outputs

- Per-ion CCS values per platform/method (e.g. ccs_values_table.csv)
- Relative differences versus manually determined vendor-software CCS values (Table 1 comparison)
- Log files from each mode run

## How to apply

Install AutoCCS from github.com/PNNL-Comp-Mass-Spec/AutoCCS via `conda env create -f environment.yml` then `conda activate autoccs`, or with Python 3.7+ and `pip install -r requirements.txt`. For stepped-field DTIMS-MS, run autoCCS.py in `--mode multi` with a TargetList.csv, an autoCCS_config.xml, frame metadata and feature CSVs from MassIVE MSV000085979, and `--threshold_n_fields 5` so CCS is computed only when at least 5 fields are observed. For single-field DTIMS or TWIMS data, run in `--mode single --single_mode batch` supplying a calibrant file (e.g. TuneMix-CCS.txt) with `--tunemix_sample_type AgilentTuneMix` and sample metadata column names; choose `--degree 1` (linear) for RapidFire-DTIMS and Bruker timsTOF, `--degree 2 --calib_method poly --ppm 150` for SLIM-based IMS, or `--calib_method power` for the linearized power function on Waters Synapt traveling-wave data. Aggregate the per-ion CCS values across platforms and compute relative differences against manually determined vendor-software values (e.g. Agilent IM-MS Browser) as reported in Table 1 / Section 3.1.

## Related tools

- **AutoCCS** (Primary software that computes CCS values from IMS-MS measurements in multi (stepped-field) or single-field batch modes) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python 3.7** (Runtime environment in which autoCCS.py is executed)
- **conda** (Recommended installer that creates the autoccs environment from environment.yml)
- **pip** (Alternative installer of dependencies via requirements.txt)

## Examples

```
python -u autoCCS.py --target_list_file data/SteppedField-DTIMS/TargetList.csv --config_file data/SteppedField-DTIMS/autoCCS_config.xml --framemeta_files "data/SteppedField-DTIMS/ImsMetadata/*.txt" --feature_files "data/SteppedField-DTIMS/Features_csv/*.csv" --output_dir data/SteppedField-DTIMS/Results/ --threshold_n_fields 5 --mode multi
```

## Evaluation signals

- CCS determinations for Agilent tune-mix ions show excellent reproducibility across the five IMS systems
- AutoCCS-calculated CCS values agree with manually determined Agilent IM-MS Browser values with acceptable relative differences (Table 1 / Section 3.1)
- In stepped-field mode, every reported CCS is derived from at least --threshold_n_fields 5 field observations
- SLIM calibration used --ppm 150 tolerance and a degree-2 polynomial; log files confirm the chosen --calib_method (poly or power) ran without error
- Output ccs_values_table.csv contains per-ion CCS values and relative differences for all targets in TargetList.csv

## Limitations

- Python 3.7 or newer is required; conda (environment.yml) is the recommended installation path
- Single-field and traveling-wave modes require a calibrant file with reference CCS values (e.g. Agilent tune-mix); stepped-field mode does not
- Discussion section content is absent from the source document, so limitations, missing artifacts, or reproducibility gaps cannot be extracted from it

## Evidence

- [abstract] We present AutoCCS, an open-source software to rapidly determine CCS values from IMS-MS measurements.: "We present AutoCCS, an open-source software to rapidly determine CCS values from IMS-MS measurements."
- [readme] Demo dataset is publicly available at MassIVE: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979](https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000085979))"
- [methods] we demonstrated the CCS determination using AutoCCS for the Agilent tune-mix samples in three different platforms: "we demonstrated the CCS determination using AutoCCS for the Agilent tune-mix samples in three different platforms: stepped-field DTIMS-MS"
- [readme] Users are allowed to apply high-order polynomial functions: quadratic (--degree 2), cubic (--degree 3), quartic (--degree 4), and so on.: "Users are allowed to apply high-order polynomial functions: quadratic (`--degree 2`), cubic (`--degree 3`), quartic (`--degree 4`), and so on."
