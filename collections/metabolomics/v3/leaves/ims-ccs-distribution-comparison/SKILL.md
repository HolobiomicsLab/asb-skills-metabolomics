---
name: ims-ccs-distribution-comparison
description: Use when running AutoCCS single-field DTIMS calibration on large multi-run
  datasets (e.g. 7106 RapidFire SPE-IMS-MS runs) and you need to evaluate whether
  per-run pressure/temperature corrections reduce CCS variance for a target ion such
  as protonated rotenone.
when_to_use_negative:
- Data are stepped-field DTIMS or TWIMS/SLIM measurements where multi-field or traveling-wave
  AutoCCS modes, not single-field calibration comparison, are appropriate
- The goal is absolute CCS determination of calibrants rather than assessing calibration-variant
  stability of an unknown analyte across many runs
- Only a single run or a handful of runs is available — %RSD comparison across runs
  is not meaningful at small n
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Computes calibrated CCS values in single-field batch mode with linear calibration
    for both the conventional and enhanced (pressure/temperature-corrected) variants
    whose distributions are compared
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Parses AutoCCS result outputs and computes per-run mean CCS and %RSD for the
    target ion
  repo: https://www.python.org/downloads/
- name: conda
  role: Installs the AutoCCS runtime environment from environment.yml
  repo: https://docs.anaconda.com/anaconda/install/
- name: pip
  role: Alternative installer for AutoCCS dependencies via requirements.txt
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:74386d7986344f04e207e153201a6cbf87935287f65b54e84fb9fe3f071d3ccd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# ims-ccs-distribution-comparison

## Summary

Compares per-run CCS distributions of a target ion across calibration variants (conventional vs enhanced single-field DTIMS calibration) by computing mean CCS and %RSD, to quantify how per-run pressure and temperature corrections stabilize CCS values in large-scale RapidFire SPE-IMS-MS studies.

## When to use

Use when running AutoCCS single-field DTIMS calibration on large multi-run datasets (e.g. 7106 RapidFire SPE-IMS-MS runs) and you need to evaluate whether per-run pressure/temperature corrections reduce CCS variance for a target ion such as protonated rotenone.

## When NOT to use

- Data are stepped-field DTIMS or TWIMS/SLIM measurements where multi-field or traveling-wave AutoCCS modes, not single-field calibration comparison, are appropriate
- The goal is absolute CCS determination of calibrants rather than assessing calibration-variant stability of an unknown analyte across many runs
- Only a single run or a handful of runs is available — %RSD comparison across runs is not meaningful at small n

## Inputs

- SPE-IMS-MS feature files (*.csv) from MassIVE MSV000085979
- IMS frame-metadata files (*.txt) with per-run pressure and temperature
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, IonPolarity columns
- Agilent tune-mix calibrant CCS list (TuneMix-CCS.txt)
- AutoCCS result outputs from conventional and enhanced single-field calibration runs

## Outputs

- Per-run CCS values for the target ion extracted from each AutoCCS result output
- Comparison table of CCS distributions (mean CCS and %RSD) for conventional vs enhanced calibration
- Validation report matching published benchmarks (194.7 ± 0.4% RSD enhanced vs 195.6 ± 1.4% RSD conventional for rotenone)

## How to apply

Obtain SPE-IMS-MS plant-extract feature files, IMS frame-metadata files, sample metadata, and the Agilent tune-mix calibrant CCS list from MassIVE MSV000085979. Run AutoCCS twice in single-field batch mode with a linear calibration function (--degree 1): once with the conventional single-field method (Kurulugama et al., 2015) and once with the enhanced calibration that incorporates per-run pressure and temperature. Parse both result outputs and extract per-run CCS values for the target ion (protonated rotenone). Compute mean CCS and percent relative standard deviation (%RSD) across runs for each variant and assemble the two distributions into a comparison table; the enhanced calibration should collapse rotenone from 195.6 (1.4% RSD) to a near-single distribution at 194.7 (0.4% RSD).

## Related tools

- **AutoCCS** (Computes calibrated CCS values in single-field batch mode with linear calibration for both the conventional and enhanced (pressure/temperature-corrected) variants whose distributions are compared) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Parses AutoCCS result outputs and computes per-run mean CCS and %RSD for the target ion) — https://www.python.org/downloads/
- **conda** (Installs the AutoCCS runtime environment from environment.yml) — https://docs.anaconda.com/anaconda/install/
- **pip** (Alternative installer for AutoCCS dependencies via requirements.txt)

## Examples

```
python autoCCS.py --config_file autoCCS_config.xml --framemeta_files "ImsMetadata/*.txt" --feature_files "Features_csv/*.csv" --calibrant_file TuneMix-CCS.txt --output_dir Results/ --tunemix_sample_type AgilentTuneMix --degree 1 --single_mode batch --mode single
```

## Evaluation signals

- Enhanced-calibration rotenone CCS mean ≈ 194.7 with ~0.4% RSD, matching Figure 3 of the source paper
- Conventional-calibration rotenone CCS mean ≈ 195.6 with ~1.4% RSD, confirming the baseline before correction
- Enhanced-calibration CCS distribution should appear almost single (near-Gaussian) rather than spread across runs
- Per-run CCS values must be extractable for every run in the 7106-run RapidFire dataset from both AutoCCS result outputs
- The %RSD of the enhanced variant must be substantially lower than the conventional variant (roughly 3-fold reduction here)

## Limitations

- Validation targets are specific to protonated rotenone in RapidFire SPE-IMS-MS plant-extract data; other ions or platforms may respond differently to pressure/temperature correction
- Discussion section content is absent from the indexed article, so reproducibility gaps and edge-case failure modes (e.g. missing frame-metadata pressure/temperature fields) cannot be assessed from context
- Requires per-run pressure and temperature metadata in the IMS frame-metadata files; the enhanced calibration cannot be applied without them

## Evidence

- [full_text] the proposed enhanced single-field method, which accounts for per-run pressure and temperature, provided a small calibrated CCS variance (194.7 with 0.4% RSD) with an almost single distribution: [evidence span withheld — anchor status: not_found]
- [readme] AutoCCS single-field batch command: "python autoCCS.py --config_file data/SingleField-RapidFire-DTIMS/autoCCS_config.xml --framemeta_files "data/SingleField-RapidFire-DTIMS/ImsMetadata/*.txt""
- [abstract] AutoCCS supports single-field methods for DTIMS: [evidence span withheld — anchor status: not_found]
- [readme] Demo dataset MassIVE accession: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
- [methods] single-field method reference: "Single field method [(Kurulugama,R.T. et al., 2015)]"
