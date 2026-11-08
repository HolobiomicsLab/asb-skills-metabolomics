---
name: ion-arrival-time-field-regression
description: Use when IMS-MS data were acquired in stepped-field DTIMS mode (e.g.,
  Agilent 6560 with multiple electric fields per ion) and per-ion CCS values must
  be computed from arrival-time vs P/V regressions rather than single-field calibration.
when_to_use_negative:
- Data were acquired in single-field DTIMS mode (e.g., RapidFire-DTIMS) or on traveling-wave
  instruments (Bruker timsTOF, Waters Synapt, SLIM) — those require AutoCCS's single-field
  or TWIMS calibration modes instead
- Ions are observed across fewer than the required number of electric fields (below
  the --threshold_n_fields 5 cutoff) and cannot support a stable arrival-time vs P/V
  regression
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AutoCCS
  role: 'Executes the stepped-field analysis: per-field arrival-time vs P/V fitting
    per Stow et al. 2017 to derive drift times and compute CCS for each ion/adduct
    in multi (--mode multi) batch processing'
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python 3.7
  role: Runtime for autoCCS.py; environment installed via conda (environment.yml)
    or pip (requirements.txt)
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:e9d46ca42e4323c5b84786330097283b45c3a6cd02fb5547c8da56e59cf55962
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ion-arrival-time-field-regression

## Summary

Determines ion drift times and collision cross sections (CCS) from stepped-field DTIMS-MS data by regressing arrival time versus electric field (P/V) across multiple field steps, following Stow et al. 2017, as automated in AutoCCS.

## When to use

Use when IMS-MS data were acquired in stepped-field DTIMS mode (e.g., Agilent 6560 with multiple electric fields per ion) and per-ion CCS values must be computed from arrival-time vs P/V regressions rather than single-field calibration.

## When NOT to use

- Data were acquired in single-field DTIMS mode (e.g., RapidFire-DTIMS) or on traveling-wave instruments (Bruker timsTOF, Waters Synapt, SLIM) — those require AutoCCS's single-field or TWIMS calibration modes instead
- Ions are observed across fewer than the required number of electric fields (below the --threshold_n_fields 5 cutoff) and cannot support a stable arrival-time vs P/V regression

## Inputs

- TargetList.csv target ion list (m/z and adduct assignments)
- autoCCS_config.xml stepped-field configuration file
- ImsMetadata frame-metadata text files (globs like data/SteppedField-DTIMS/ImsMetadata/*.txt)
- Features_csv feature tables with per-field arrival times (data/SteppedField-DTIMS/Features_csv/*.csv)

## Outputs

- Per-ion/adduct CCS results table with regression fit statistics in data/SteppedField-DTIMS/Results/
- Run log capturing completion status (data/SteppedField-DTIMS/LogFiles/multi.log)

## How to apply

Collect frame-metadata text files (ImsMetadata) and per-feature arrival-time tables (Features_csv) covering ions measured across several electric fields on a drift tube instrument such as the Agilent 6560. Run AutoCCS with a target list (TargetList.csv) and XML configuration (autoCCS_config.xml), setting --threshold_n_fields 5 so an ion is only processed if observed across at least five fields; fewer-field ions are excluded because the linear arrival-time vs P/V fit (Stow et al. 2017 stepped field method) is unreliable with too few points. AutoCCS fits drift time per ion/adduct from the regression and converts it to a CCS, writing per-ion CCS results with regression fit statistics into the output directory (Results/) and logging the run to a log file. Validate by checking the run log for successful completion and confirming each qualifying target ion reports a computed CCS with fit statistics, while sub-threshold ions are absent.

## Related tools

- **AutoCCS** (Executes the stepped-field analysis: per-field arrival-time vs P/V fitting per Stow et al. 2017 to derive drift times and compute CCS for each ion/adduct in multi (--mode multi) batch processing) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python 3.7** (Runtime for autoCCS.py; environment installed via conda (environment.yml) or pip (requirements.txt)) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS

## Examples

```
python -u autoCCS.py --target_list_file data/SteppedField-DTIMS/TargetList.csv --config_file data/SteppedField-DTIMS/autoCCS_config.xml --framemeta_files "data/SteppedField-DTIMS/ImsMetadata/*.txt" --feature_files "data/SteppedField-DTIMS/Features_csv/*.csv" --output_dir data/SteppedField-DTIMS/Results/ --threshold_n_fields 5 --mode multi
```

## Evaluation signals

- Run log (multi.log) records successful completion of the --mode multi stepped-field run
- Results table reports a computed CCS with regression fit statistics for each qualifying target ion
- Ions measured across fewer than 5 electric fields (--threshold_n_fields 5) are excluded from the results
- Computed CCS values differ from vendor software / manually determined CCS by under 0.2% for most tune-mix ions
- CCS results reproduce across IMS systems, consistent with the paper's reported reproducibility

## Limitations

- Requires ions to be measured across at least --threshold_n_fields (demo uses 5) electric fields; sparsely sampled ions are dropped
- Applicable to drift tube (DTIMS) stepped-field acquisitions only; traveling-wave and SLIM data need different AutoCCS calibration methods
- Discussion-section limitations were not extractable from the provided context (no changelog/limitations found in the repository README)

## Evidence

- [abstract] AutoCCS supports stepped-field methods for drift tube-based IMS (DTIMS): "We conducted various IMS experiments in different formats to demonstrate the flexibility of AutoCCS for automated CCS calculation: (i) stepped-field methods for drift tube-based IMS (DTIMS)"
- [methods] Stepped field method reference: "Stepped field method [(Stow,S.M. et al., 2017)]"
- [methods] threshold_n_fields CLI flag: "--threshold_n_fields 5"
- [readme] README stepped-field command: "python -u autoCCS.py --target_list_file data/SteppedField-DTIMS/TargetList.csv --config_file data/SteppedField-DTIMS/autoCCS_config.xml"
- [methods] Demo dataset availability: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
