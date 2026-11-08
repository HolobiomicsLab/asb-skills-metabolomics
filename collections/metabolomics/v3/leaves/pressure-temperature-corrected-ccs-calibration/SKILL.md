---
name: pressure-temperature-corrected-ccs-calibration
description: Use when single-field DTIMS CCS values for the same ion vary across many
  runs (e.g. large RapidFire SPE-IMS-MS batches) and per-run pressure/temperature
  fluctuations are suspected of inflating %RSD.
when_to_use_negative:
- Data were acquired with stepped-field DTIMS, where CCS is derived directly from
  multi-field measurements (--mode multi) rather than calibrated single-field drift
  times
- The platform is TWIMS/SLIM and requires traveling-wave calibration (e.g. linearized
  power function) rather than single-field drift-time calibration
- Only a single run is available, so per-run pressure/temperature correction cannot
  reduce run-to-run variance
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Runs the single-field calibration with per-run pressure/temperature corrections
    on RapidFire SPE-IMS-MS feature and frame-metadata files
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Runtime for executing autoCCS.py and parsing the per-run CCS result outputs
    for RSD analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:b6f1e5885009fe67846d83ea0abec6f7d7e761dd3f0a90318c58a0c440e81563
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
yes_rate: 0.7
n_claims: 10
---

# pressure-temperature-corrected-ccs-calibration

## Summary

An enhanced single-field DTIMS calibration implemented in AutoCCS that incorporates per-run pressure and temperature from IMS frame metadata into the CCS calculation, reducing calibrated CCS variance caused by instrument condition fluctuations in large-scale RapidFire SPE-IMS-MS runs.

## When to use

Use when single-field DTIMS CCS values for the same ion vary across many runs (e.g. large RapidFire SPE-IMS-MS batches) and per-run pressure/temperature fluctuations are suspected of inflating %RSD.

## When NOT to use

- Data were acquired with stepped-field DTIMS, where CCS is derived directly from multi-field measurements (--mode multi) rather than calibrated single-field drift times
- The platform is TWIMS/SLIM and requires traveling-wave calibration (e.g. linearized power function) rather than single-field drift-time calibration
- Only a single run is available, so per-run pressure/temperature correction cannot reduce run-to-run variance

## Inputs

- IMS frame-metadata files (per-run pressure and temperature records, e.g. .txt from the ImsMetadata directory)
- SPE-IMS-MS feature files (Features_csv/*.csv)
- Sample metadata table (Datasets.csv with SampleType, RawFileName, IonPolarity columns)
- Calibrant CCS list (TuneMix-CCS.txt for Agilent tune-mix)
- AutoCCS XML configuration file (autoCCS_config.xml)

## Outputs

- AutoCCS result files with per-run calibrated CCS values for each calibration variant
- Comparison table of mean CCS and %RSD across runs for conventional vs enhanced calibration
- CCS distributions per variant (e.g. rotenone at 194.7 with 0.4% RSD enhanced vs 195.6 with 1.4% RSD conventional)

## How to apply

Download the SPE-IMS-MS feature files, IMS frame-metadata files, sample metadata table, and calibrant CCS list from MassIVE MSV000085979. Run AutoCCS twice in single-field mode (--mode single, --single_mode batch) with Agilent tune-mix as calibrant (--tunemix_sample_type AgilentTuneMix, --calibrant_file TuneMix-CCS.txt) and a linear calibration function (--degree 1): once with the conventional single-field method (Kurulugama et al., 2015) and once with the enhanced single-field calibration that folds per-run pressure and temperature corrections into the CCS calculation. Parse both AutoCCS result outputs and extract per-run CCS values for the target ion (e.g. protonated rotenone). Compute mean CCS and %RSD per calibration variant and compare distributions; the enhanced method should collapse the distribution (for rotenone: mean 194.7, 0.4% RSD vs conventional 195.6, 1.4% RSD).

## Related tools

- **AutoCCS** (Runs the single-field calibration with per-run pressure/temperature corrections on RapidFire SPE-IMS-MS feature and frame-metadata files) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Runtime for executing autoCCS.py and parsing the per-run CCS result outputs for RSD analysis)

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt" --feature_files "data/SingleField-RapidFire-DTIMS/Features_csv/*.csv" --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --output_dir data/SingleField-RapidFire-DTIMS/Results/ --tunemix_sample_type AgilentTuneMix --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- Enhanced-calibration rotenone CCS distribution matches mean 194.7 with 0.4% RSD versus conventional 195.6 with 1.4% RSD
- Calibrated CCS %RSD across the 7106 RapidFire runs is lower for the enhanced variant than the conventional single-field variant
- The enhanced-calibration CCS distribution is almost single (unimodal) rather than split by run conditions
- Both calibration variants use identical inputs (Agilent tune-mix calibrant, --degree 1 linear function, same feature and frame-metadata files), so differences are attributable to the pressure/temperature correction

## Limitations

- Demonstrated for protonated rotenone on the RapidFire SPE-IMS-MS platform; variance reduction magnitude may differ for other ions, ionization modes, or collision gases
- Requires IMS frame metadata containing per-run pressure and temperature; runs lacking these records cannot benefit from the enhanced correction
- Single-field calibration still relies on calibrant CCS values (Agilent tune-mix) and a chosen calibration function (linear, polynomial, or linearized power) rather than direct stepped-field CCS measurement

## Evidence

- [results] the proposed enhanced single-field method, which accounts for per-run pressure and temperature, provided a small calibrated CCS variance (194.7 with 0.4% RSD) with an almost single distribution: [evidence span withheld — anchor status: not_found]
- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS), single-field methods for DTIMS (standard and enhanced calibration), linear calibration: [evidence span withheld — anchor status: not_found]
- [methods] Single field method [(Kurulugama,R.T. et al., 2015)]: "Single field method [(Kurulugama,R.T. et al., 2015)]"
- [readme] --tunemix_sample_type AgilentTuneMix --sample_meta data/SingleField-RapidFire-DTIMS/Datasets.csv --colname_for_sample_type SampleType --colname_for_filename RawFileName: "--tunemix_sample_type AgilentTuneMix --sample_meta data/SingleField-RapidFire-DTIMS/Datasets.csv --colname_for_sample_type SampleType --colname_for_filename RawFileName"
