---
name: ccs-reference-value-comparison
description: Use when you have AutoCCS-computed CCS values from two or more calibration
  methods or IMS platforms and authoritative reference CCS values (e.g. DTIMS-derived
  or vendor-software values) to validate accuracy per analyte.
when_to_use_negative:
- When only a single calibration method/platform was run and no reference CCS values
  exist — there is nothing to compare against
- When the target analyte's arrival time lies outside the calibrant range (e.g. Pterin
  relative to TAA calibrants), since the comparison is confounded by extrapolation
  rather than method quality
- For stepped-field DTIMS analysis, where CCS is determined directly from physics
  rather than calibrated against reference values
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AutoCCS
  role: Generates the computed CCS values being compared, under each calibration method
    (--calib_method poly / power)
  repo: https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- name: Python
  role: Used to tabulate computed vs. reference CCS values and compute relative deviations
    (Python 3.7+ environment)
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab429
    title: 'AutoCCS: automated collision cross-section calculation software for ion
      mobility spectrometry–mass spectrometry'
  source_content_hash: sha256:2e6a9ef1362b4a7f3e137cbbe9aa62a82c813c74128873617358a82f641d7ca1
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
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ccs-reference-value-comparison

## Summary

Compares CCS values computed by AutoCCS (across calibration methods or IMS platforms, e.g. SLIM IMS-MS vs. DTIMS-MS) against reference CCS values to quantify relative deviation and select the best-performing calibration approach.

## When to use

Use when you have AutoCCS-computed CCS values from two or more calibration methods or IMS platforms and authoritative reference CCS values (e.g. DTIMS-derived or vendor-software values) to validate accuracy per analyte.

## When NOT to use

- When only a single calibration method/platform was run and no reference CCS values exist — there is nothing to compare against
- When the target analyte's arrival time lies outside the calibrant range (e.g. Pterin relative to TAA calibrants), since the comparison is confounded by extrapolation rather than method quality
- For stepped-field DTIMS analysis, where CCS is determined directly from physics rather than calibrated against reference values

## Inputs

- AutoCCS Results output tables containing computed CCS values from multiple calibration runs (e.g. poly vs. power)
- Reference CCS values (e.g. DTIMS reference values from Table 3, or manually determined Agilent IM-MS Browser values)
- Per-analyte identifiers (e.g. IROA standards: Adenine, Trehalose, Pterin; or Agilent tune-mix ions)

## Outputs

- Per-analyte table of computed CCS values per calibration method/platform alongside reference CCS values
- Relative deviation (%) per analyte per calibration method (e.g. ccs_values_table.csv)
- Decision on which calibration method (polynomial vs. linearized power) better matches the reference values

## How to apply

Run AutoCCS under each calibration configuration to be compared — e.g. SLIM IMS-MS single-field batch mode once with quadratic polynomial calibration (--calib_method poly --degree 2 --ppm 150) and once with the linearized power function (--calib_method power), each calibrated with the AgilentTuneMix calibrants in TuneMix-CCS_POS.txt. Extract the computed CCS values for the analytes of interest (e.g. the IROA standards Adenine, Trehalose, Pterin) from the Results output of each run. Tabulate each computed CCS against its reference value (e.g. the DTIMS-MS values from Table 3, or manual Agilent IM-MS Browser values) and compute the relative deviation per analyte per method. Select the calibration method with lower overall deviation; investigate individual outliers — a notably high deviation (e.g. Pterin's 2.26%) may indicate the analyte's arrival time falls outside the TAA calibrant range. Judge success when computed CCS values agree with reference values within the accuracy the paper reports for AutoCCS across platforms.

## Related tools

- **AutoCCS** (Generates the computed CCS values being compared, under each calibration method (--calib_method poly / power)) — https://github.com/PNNL-Comp-Mass-Spec/AutoCCS
- **Python** (Used to tabulate computed vs. reference CCS values and compute relative deviations (Python 3.7+ environment))

## Examples

```
python autoCCS.py --config_file data/SLIM-IMS/autoCCS_config.xml --framemeta_files "data/SLIM-IMS/ImsMetadata/*.txt" --feature_files "data/SLIM-IMS/Features_csv/*.csv" --calibrant_file data/SLIM-IMS/TuneMix-CCS_POS.txt --sample_meta data/SLIM-IMS/Datasets.csv --calib_method power --ppm 150 --mode single --single_mode batch
```

## Evaluation signals

- Output table exists with CCS columns for both calibration methods (poly and power)
- Computed CCS values agree with DTIMS reference values within the accuracy the paper reports for AutoCCS across platforms
- Relative deviations per analyte are tabulated and finite (no missing values for IROA standards Adenine, Trehalose, Pterin)
- Across five IMS platforms, AutoCCS tune-mix CCS values show acceptable differences versus manually determined vendor-software values
- Outlier deviations are explainable — e.g. Pterin's 2.26% error attributed to its arrival time falling outside the TAA calibrant range

## Limitations

- Analytes whose arrival time falls outside the calibrant range yield elevated errors (Pterin error was notably higher, 2.26%, likely because its arrival time fell outside the TAA calibrant range)
- The linearized power regression agreed better with DTIMS reference values than polynomial regression for SLIM IMS-MS, but this preference may not generalize to other platforms or calibrant sets
- Discussion/limitations content is absent from the available article text, so further caveats could not be extracted

## Evidence

- [other] the linearized power regression provided CCS values more similar to DTIMS than the polynomial regression, though the Pterin error was notably higher (2.26%), likely because its arrival time fell outside the TAA calibrant range.: [evidence span withheld — anchor status: not_found]
- [abstract] AutoCCS offers an accurate and reproducible determination of CCS for both standard and unknown analyte ions in various IMS-MS platforms, IMS-field methods, ionization modes and collision gases: "We demonstrated that AutoCCS offers an accurate and reproducible determination of CCS for both standard and unknown analyte ions in various IMS-MS platforms"
- [other] CCS determinations showed excellent reproducibility across the five IMS systems, and AutoCCS automatically calculated CCS values with acceptable differences compared to manually determined CCS values: [evidence span withheld — anchor status: not_found]
- [readme] Calibration functions for single field and traveling wave-based methods - Linear function, Polynomial functions (e.g. quadratic or cubic functions), Linearized power function: "Calibration functions for single field and traveling wave-based methods
  - Linear function
  - Polynomial functions (e.g. quadratic or cubic functions)
  - Linearized power function"
- [readme] Demo dataset is publicly available at MassIVE (Accession: MSV000085979): "Demo dataset is publicly available at MassIVE (Accession: [MSV000085979](https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000085979))"
