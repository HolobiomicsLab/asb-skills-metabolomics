---
name: cross-run-ccs-reproducibility-assessment
description: Use when a large batch of IMS-MS runs (e.g. 7106 RapidFire SPE-IMS-MS
  runs) yields a repeat analyte measurement and you need to quantify CCS variance
  across runs, or to compare calibration variants (conventional vs. pressure/temperature-corrected
  single-field) by their %RSD.
when_to_use_negative:
- The data are stepped-field DTIMS measurements, where CCS is computed directly per
  field rather than via single-field cross-run calibration reproducibility
- The platform is TWIMS/SLIM or Bruker timsTOF, where traveling-wave or timsTOF calibration
  functions apply instead of the DTIMS single-field method
- Only a single run or dataset is available — %RSD across runs is undefined without
  a multi-run batch
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Computes calibrated per-run CCS values in single-field batch mode for both
    the conventional and enhanced (pressure/temperature-corrected) calibration variants
    whose reproducibility is compared
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Runtime for autoCCS.py and for parsing result outputs to compute mean CCS
    and %RSD across runs
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:2af81fec7fb5ecfa54aa9ee24ca94b3579d0f3a0542a4033711ce2b6e3eb7621
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-run-ccs-reproducibility-assessment

## Summary

Quantify how consistently an analyte's collision cross section (CCS) is reproduced across many acquisition runs by computing per-run CCS distributions, mean CCS, and %RSD — here applied to protonated rotenone across 7106 RapidFire SPE-IMS-MS plant-extract runs processed with AutoCCS.

## When to use

Use when a large batch of IMS-MS runs (e.g. 7106 RapidFire SPE-IMS-MS runs) yields a repeat analyte measurement and you need to quantify CCS variance across runs, or to compare calibration variants (conventional vs. pressure/temperature-corrected single-field) by their %RSD.

## When NOT to use

- The data are stepped-field DTIMS measurements, where CCS is computed directly per field rather than via single-field cross-run calibration reproducibility
- The platform is TWIMS/SLIM or Bruker timsTOF, where traveling-wave or timsTOF calibration functions apply instead of the DTIMS single-field method
- Only a single run or dataset is available — %RSD across runs is undefined without a multi-run batch

## Inputs

- SPE-IMS-MS plant-extract feature files (Features_csv/*.csv)
- IMS frame-metadata files (ImsMetadata/*.txt) containing per-run pressure and temperature
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns
- Calibrant CCS list (TuneMix-CCS.txt) for Agilent tune-mix
- AutoCCS config file (autoCCS_config.xml) and target list for the analyte of interest (e.g. rotenone)

## Outputs

- Per-run calibrated CCS values for the target ion from each calibration variant
- Mean CCS and %RSD of the analyte across all runs per variant
- Comparison table of conventional vs. enhanced-calibration CCS distributions

## How to apply

Run AutoCCS in single-field batch mode over the feature files and IMS frame-metadata files, calibrating with Agilent tune-mix (TuneMix-CCS.txt) and a linear function (--degree 1), first with the conventional single-field method (Kurulugama et al., 2015) and again with the enhanced single-field calibration that incorporates per-run pressure and temperature. Parse both AutoCCS result outputs and extract the per-run CCS values for the target ion (e.g. protonated rotenone). For each calibration variant, compute the mean CCS and percent relative standard deviation (%RSD) across runs and assemble the two CCS distributions into a comparison table. A calibration that reduces condition-induced spread is judged better by a lower %RSD and a tighter, more unimodal distribution — the enhanced method gives mean 194.7 with 0.4% RSD versus the conventional 195.6 with 1.4% RSD for rotenone.

## Related tools

- **AutoCCS** (Computes calibrated per-run CCS values in single-field batch mode for both the conventional and enhanced (pressure/temperature-corrected) calibration variants whose reproducibility is compared) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Runtime for autoCCS.py and for parsing result outputs to compute mean CCS and %RSD across runs) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS

## Examples

```
python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt" --feature_files "data/SingleField-RapidFire-DTIMS/Features_csv/*.csv" --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --output_dir data/SingleField-RapidFire-DTIMS/Results/ --tunemix_sample_type AgilentTuneMix --sample_meta data/SingleField-RapidFire-DTIMS/Datasets.csv --colname_for_sample_type SampleType --colname_for_filename RawFileName --colname_for_ionization IonPolarity --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- Enhanced-calibration %RSD of rotenone CCS across the 7106 RapidFire runs falls near the reported 0.4% (vs. 1.4% conventional)
- Mean calibrated CCS matches the reported 194.7 (enhanced) and 195.6 (conventional) for protonated rotenone
- The enhanced-calibration CCS distribution is nearly a single distribution (unimodal) rather than spread across condition-induced clusters
- AutoCCS single-field batch run completes with --mode single --single_mode batch --degree 1 using AgilentTuneMix calibrant and produces result outputs for all runs

## Limitations

- Reproducibility gains depend on frame metadata containing per-run pressure and temperature; without these the enhanced correction cannot be applied
- Single-field CCS is calibrated relative to Agilent tune-mix reference CCS values, so errors in calibrant CCS propagate to all reported values
- The article's discussion section is absent from the available context, so platform-specific caveats and generalization limits cannot be verified

## Evidence

- [results] the proposed enhanced single-field method, which accounts for per-run pressure and temperature, provided a small calibrated CCS variance (194.7 with 0.4% RSD): [evidence span withheld — anchor status: not_found]
- [methods] Single field method [(Kurulugama,R.T. et al., 2015)]: "Single field method [(Kurulugama,R.T. et al., 2015)]"
- [abstract] an open-source software to rapidly determine CCS values from IMS-MS measurements: "We present AutoCCS, an open-source software to rapidly determine CCS values from IMS-MS measurements."
- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS), single-field methods for DTIMS (standard and enhanced calibration), linear calibration: [evidence span withheld — anchor status: not_found]
- [readme] python autoCCS.py --calibrant_file data/SingleField-RapidFire-DTIMS/TuneMix-CCS.txt --tunemix_sample_type AgilentTuneMix --degree 1 --single_mode batch --mode single: "python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt" --feature_files"
