---
name: linearized-power-function-ccs-regression
description: Use when converting arrival-time measurements from single-field DTIMS
  or traveling-wave/SLIM IMS-MS into CCS values and the calibrant-to-analyte relationship
  is non-linear — e.g. SLIM helium data calibrated with TAA/tune-mix references where
  a quadratic poly fit underperforms.
when_to_use_negative:
- Data are stepped-field DTIMS-MS measurements — AutoCCS handles these in multi-field
  mode where calibration functions are not applied this way
- Input is raw UIMF or MS frames rather than extracted feature tables — AutoCCS expects
  Features_csv files, not UIMF-Library output
- Analyte arrival times lie outside the calibrant range (e.g. Pterin outside the TAA
  calibrant range), where extrapolation error dominates the choice of calibration
  function
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AutoCCS
  role: Executes the linearized power regression calibration of arrival times to CCS
    values via --calib_method power in single-field batch mode
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: conda
  role: Creates the autoccs environment from environment.yml to run AutoCCS with Python
    3.7+ dependencies
  repo: https://docs.anaconda.com/anaconda/install/
- name: pip
  role: Alternative installation of AutoCCS dependencies via requirements.txt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:7b27bb3288caff1dabdf2ce5e94e705921e58a831a71e0e846d560e052e36524
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
  tools:
  - AutoCCS
  - Python
  - conda
  - pip
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# linearized-power-function-ccs-regression

## Summary

Calibrates single-field or traveling-wave IMS-MS arrival times to collision cross-section (CCS) values in AutoCCS using the linearized power regression function (Ruotolo et al., 2008), as an alternative to linear or polynomial (quadratic/cubic) calibration functions.

## When to use

Use when converting arrival-time measurements from single-field DTIMS or traveling-wave/SLIM IMS-MS into CCS values and the calibrant-to-analyte relationship is non-linear — e.g. SLIM helium data calibrated with TAA/tune-mix references where a quadratic poly fit underperforms.

## When NOT to use

- Data are stepped-field DTIMS-MS measurements — AutoCCS handles these in multi-field mode where calibration functions are not applied this way
- Input is raw UIMF or MS frames rather than extracted feature tables — AutoCCS expects Features_csv files, not UIMF-Library output
- Analyte arrival times lie outside the calibrant range (e.g. Pterin outside the TAA calibrant range), where extrapolation error dominates the choice of calibration function

## Inputs

- Feature tables (Features_csv/*.csv) with per-dataset m/z and arrival-time measurements
- IMS metadata files (ImsMetadata/*.txt)
- Sample metadata table (Datasets.csv) with SampleType, RawFileName, and IonPolarity columns
- Calibrant CCS reference file (e.g. TuneMix-CCS_POS.txt) mapping calibrant ions to literature/reference CCS values
- AutoCCS XML config file (autoCCS_config.xml)

## Outputs

- Results directory with fitted CCS values per analyte ion for the power-function calibration run
- Relative-deviation table comparing power-derived CCS values to DTIMS reference values (e.g. Table 3 IROA standards)

## How to apply

Run AutoCCS in single-field batch mode with a calibrant reference file (e.g. TuneMix-CCS_POS.txt listing known CCS values) and select the power calibration via --calib_method power, mapping sample metadata columns (SampleType, RawFileName, IonPolarity) so calibrant samples (e.g. AgilentTuneMix) are identified automatically. The linearized power function fits corrected drift time against reference CCS following Ruotolo et al. (2008), capturing the non-linear trend typical of traveling-wave and SLIM separations in helium. Compare against a polynomial run (--calib_method poly --degree 2) by tabulating relative deviations of computed CCS values from independent DTIMS reference values (e.g. IROA standards Adenine, Trehalose, Pterin). Choose the calibration whose computed CCS values agree better with the DTIMS references; note that analytes whose arrival times fall outside the calibrant range (e.g. Pterin, 2.26% error) can inflate errors regardless of the fitted function.

## Related tools

- **AutoCCS** (Executes the linearized power regression calibration of arrival times to CCS values via --calib_method power in single-field batch mode) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **conda** (Creates the autoccs environment from environment.yml to run AutoCCS with Python 3.7+ dependencies) — https://docs.anaconda.com/anaconda/install/
- **pip** (Alternative installation of AutoCCS dependencies via requirements.txt)

## Examples

```
python autoCCS.py --config_file data/SingleField-SLIM/autoCCS_config.xml --framemeta_files "data/SingleField-SLIM/ImsMetadata/*.txt" --feature_files "data/SingleField-SLIM/Features_csv/*.csv" --calibrant_file data/SingleField-SLIM/TuneMix-CCS_POS.txt --output_dir data/SingleField-SLIM/Results_power/ --tunemix_sample_type AgilentTuneMix --sample_meta data/SingleField-SLIM/Datasets.csv --colname_for_sample_type SampleType --colname_for_filename RawFileName --colname_for_ionization IonPolarity --calib_method power --single_mode batch --mode single
```

## Evaluation signals

- Results output table exists with computed CCS columns for the power-calibration run
- Computed CCS values for dual-platform standards (Adenine, Trehalose, Pterin) agree with DTIMS reference values within reported AutoCCS cross-platform accuracy
- Power-function deviations from DTIMS references are smaller than the polynomial (--calib_method poly --degree 2) deviations for the same standards
- Known outlier expected: Pterin error around 2.26% because its arrival time falls outside the TAA calibrant range
- Calibrant samples are correctly identified by SampleType=AgilentTuneMix mapping in the metadata columns

## Limitations

- CCS error is inflated when the analyte's arrival time falls outside the TAA calibrant range (Pterin at 2.26%)
- Linearized power is only one of the supported single-field/traveling-wave calibration functions (linear, polynomial quadratic/cubic); the best choice is platform- and dataset-dependent
- The article's discussion/limitations section is absent from the extracted document, so broader caveats cannot be traced

## Evidence

- [methods] Linearized power function calibration follows Ruotolo et al. 2008: "Linearized power function [(Ruotolo,B.T. et al., 2008)]"
- [readme] AutoCCS supports polynomial and linearized power calibration functions for traveling wave methods: "Calibration functions for single field and traveling wave-based methods"
- [results] Power regression gave CCS values closer to DTIMS than polynomial, but Pterin erred at 2.26% due to being outside the calibrant range: [evidence span withheld — anchor status: not_found]
- [methods] Demo SLIM data available at MassIVE MSV000085979: "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979]"
