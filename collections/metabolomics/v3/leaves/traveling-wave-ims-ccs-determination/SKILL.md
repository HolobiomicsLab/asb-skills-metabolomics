---
name: traveling-wave-ims-ccs-determination
description: Use when processing traveling-wave or SLIM IMS-MS feature data (Features_csv
  + ImsMetadata + sample metadata) where arrival times must be converted to CCS via
  calibration against TAA/TuneMix reference CCS values.
when_to_use_negative:
- Data are stepped-field DTIMS measurements with multiple electric fields — use AutoCCS
  --mode multi (stepped field method) instead of traveling-wave calibration
- No calibrant ions with trusted reference CCS values are present in the run (calibrant
  range must bracket analyte arrival times)
- The goal is raw arrival-time extraction or drift-time deconvolution rather than
  CCS calculation from already-extracted features
tools:
- name: AutoCCS
  role: Executes single-field batch CCS determination with traveling-wave calibration
    functions (polynomial or linearized power) on SLIM IMS-MS feature data
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: conda
  role: Creates the autoccs environment from environment.yml to install dependencies
- name: pip
  role: Installs Python 3.7+ dependencies from requirements.txt when conda is not
    used
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:a8f7804a70b07bb1c15796a2d3b2d80fa0d2d5a49509449e1e6186a3704502ea
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# traveling-wave-ims-ccs-determination

## Summary

Determines collision cross-section (CCS) values from traveling-wave / SLIM-based IMS-MS data by calibrating measured arrival times against reference CCS values (e.g. TAA calibrants, Agilent TuneMix) with a polynomial or linearized power regression, as implemented in AutoCCS.

## When to use

Use when processing traveling-wave or SLIM IMS-MS feature data (Features_csv + ImsMetadata + sample metadata) where arrival times must be converted to CCS via calibration against TAA/TuneMix reference CCS values.

## When NOT to use

- Data are stepped-field DTIMS measurements with multiple electric fields — use AutoCCS --mode multi (stepped field method) instead of traveling-wave calibration
- No calibrant ions with trusted reference CCS values are present in the run (calibrant range must bracket analyte arrival times)
- The goal is raw arrival-time extraction or drift-time deconvolution rather than CCS calculation from already-extracted features

## Inputs

- Feature tables (Features_csv/*.csv) with per-ion arrival-time and m/z measurements from SLIM/TWIMS IMS-MS
- ImsMetadata files (frame-level metadata for SLIM datasets)
- Calibrant CCS reference file (e.g. TuneMix-CCS_POS.txt listing TAA salt reference CCS values)
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns

## Outputs

- Results table of fitted CCS values per ion per run (one set per calibration function: polynomial vs. linearized power)
- Log file recording calibration run parameters and outcomes

## How to apply

Run AutoCCS in single-field batch mode on SLIM/TWIMS feature tables, supplying ImsMetadata files, Features_csv files, a calibrant CCS reference file (e.g. TuneMix-CCS_POS.txt for the TAA salts in Agilent TuneMix), and sample metadata (Datasets.csv) mapped via --colname_for_sample_type/--colname_for_filename/--colname_for_ionization. Because traveling-wave CCS is non-linear, calibrate with either a quadratic polynomial (--calib_method poly --degree 2) or the linearized power function (--calib_method power, Ruotolo et al. 2008), using a mass tolerance (e.g. --ppm 150) to match observed calibrant ions. Run both calibration functions when comparing platforms: the linearized power regression generally yields CCS values closer to DTIMS references, but analytes whose arrival times fall outside the calibrant range (e.g. Pterin) can show higher error (~2.26%). Extract computed CCS for target ions from the Results output and tabulate relative deviations against DTIMS reference values.

## Related tools

- **AutoCCS** (Executes single-field batch CCS determination with traveling-wave calibration functions (polynomial or linearized power) on SLIM IMS-MS feature data) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **conda** (Creates the autoccs environment from environment.yml to install dependencies)
- **pip** (Installs Python 3.7+ dependencies from requirements.txt when conda is not used)

## Examples

```
python autoCCS.py --framemeta_files "data/SLIM/ImsMetadata/*.txt" --feature_files "data/SLIM/Features_csv/*.csv" --calibrant_file TuneMix-CCS_POS.txt --output_dir data/SLIM/Results/ --tunemix_sample_type AgilentTuneMix --sample_meta data/SLIM/Datasets.csv --colname_for_sample_type SampleType --colname_for_filename RawFileName --colname_for_ionization IonPolarity --calib_method power --ppm 150 --single_mode batch --mode single
```

## Evaluation signals

- Output directory contains a Results table with CCS columns for each calibration method run (poly and power)
- Computed CCS for analytes with DTIMS reference values agree within the paper's reported cross-platform accuracy
- Linearized-power CCS values deviate less from DTIMS reference values than polynomial-fitted values for shared analytes (Adenine, Trehalose, Pterin)
- Calibrant ions are matched within the specified tolerance (e.g. --ppm 150) and bracket the analyte arrival-time range; analytes outside the range (Pterin, ~2.26% error) are flagged
- Log file shows calibration completed without unmatched calibrants and the run mode (single/batch) was correctly applied

## Limitations

- CCS error rises for analytes whose arrival times fall outside the calibrant range — Pterin showed a notably higher 2.26% error, likely because its arrival time was outside the TAA calibrant range
- Linearized power regression was more similar to DTIMS than polynomial regression for the IROA standards, but choice of calibration function materially affects agreement
- Discussion/limitations section of the article is absent from the provided text, so additional caveats cannot be verified

## Evidence

- [abstract] linear calibration for Bruker timsTOF and non-linear calibration for traveling wave based-IMS in Waters Synapt and SLIM: [evidence span withheld — anchor status: not_found]
- [readme] Traveling wave based-IMS MS such as SLIM: "Traveling wave based-IMS MS (TWIMS-MS) such as Structures for Lossless Ion Manipulations (SLIM)-based IMS-MS"
- [methods] Linearized power function (Ruotolo et al., 2008): "Linearized power function [(Ruotolo,B.T. et al., 2008)]"
- [methods] Calibration functions for single field and traveling wave-based methods: "Calibration functions for single field and traveling wave-based methods"
