---
name: calibrant-reference-mass-selection
description: Use when raw m/z values extracted from an MZA file show systematic mass
  error and a list of known calibrant species must be converted into accurate reference
  m/z values before fitting mzapy.calibration.MassCalibration.
when_to_use_negative:
- Measured peaks are unknown analytes with no assigned compound formula — monoiso_mass/ms_adduct_mz
  require known compositions to generate reference masses
- The goal is CCS calibration for single-field DTIMS rather than m/z mass calibration
  — use the mzapy CCS calibration routine instead
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzapy
  role: Provides monoiso_mass, ms_adduct_formula, and ms_adduct_mz to compute reference
    calibrant masses, and MassCalibration to fit and apply the calibration
  repo: https://github.com/PNNL-m-q/mzapy
- name: mzapy.isotopes.monoiso_mass
  role: Computes exact monoisotopic neutral masses of calibrant compounds using Unimod
    element masses
  repo: https://github.com/PNNL-m-q/mzapy
- name: mzapy.isotopes.ms_adduct_mz
  role: Converts neutral monoisotopic masses to reference adduct m/z values for the
    calibrant list
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: HDF5-based file format from which measured calibrant m/z values are extracted
    via mzapy.MZA
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Underlying HDF5 access layer for reading MZA files containing the raw calibrant
    spectra
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:df4758c8d8acb5a9675440f5316f721d3efd61da979ccf30db925271c0e2214b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - Python
  - h5py
  - mzapy
  - mzapy.isotopes.monoiso_mass
  - mzapy.isotopes.ms_adduct_mz
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# calibrant-reference-mass-selection

## Summary

Builds a reference calibrant list for mass calibration by computing exact monoisotopic neutral masses and adduct m/z values with mzapy.isotopes (monoiso_mass, ms_adduct_mz), using element masses sourced from Unimod, so measured m/z values from MZA files can be paired with accurate reference masses for fitting a MassCalibration model.

## When to use

Use when raw m/z values extracted from an MZA file show systematic mass error and a list of known calibrant species must be converted into accurate reference m/z values before fitting mzapy.calibration.MassCalibration.

## When NOT to use

- Measured peaks are unknown analytes with no assigned compound formula — monoiso_mass/ms_adduct_mz require known compositions to generate reference masses
- The goal is CCS calibration for single-field DTIMS rather than m/z mass calibration — use the mzapy CCS calibration routine instead

## Inputs

- List of calibrant compound formulas/neutral masses
- Adduct type(s) observed for each calibrant (e.g., protonation)
- Measured calibrant m/z values extracted from an MZA file via mzapy.MZA

## Outputs

- Reference calibrant m/z list paired with measured m/z values (input to MassCalibration.fit)
- Residual table of pre- and post-calibration ppm mass errors per calibrant (e.g., mass_calibration_residuals.csv)

## How to apply

For each calibrant compound, compute its exact monoisotopic mass with mzapy.isotopes.monoiso_mass, which uses element masses sourced from Unimod. Convert the neutral mass to the expected ion m/z for the observed adduct with mzapy.isotopes.ms_adduct_mz (ms_adduct_formula is available to check the adduct composition). Pair each measured m/z extracted from the raw MZA file (via mzapy.MZA) with the corresponding reference calibrant m/z and pass the paired lists to mzapy.calibration.MassCalibration.fit (inherited from _CalibrationBase). Apply the fitted model with calibrated_mass/transform to obtain calibrated m/z values, then compute ppm mass errors before and after calibration for every calibrant; the reference list quality is validated by post-calibration ppm residuals falling within the tolerances enforced by the unittest.TestCase in mzapy/test.

## Related tools

- **mzapy** (Provides monoiso_mass, ms_adduct_formula, and ms_adduct_mz to compute reference calibrant masses, and MassCalibration to fit and apply the calibration) — https://github.com/PNNL-m-q/mzapy
- **mzapy.isotopes.monoiso_mass** (Computes exact monoisotopic neutral masses of calibrant compounds using Unimod element masses) — https://github.com/PNNL-m-q/mzapy
- **mzapy.isotopes.ms_adduct_mz** (Converts neutral monoisotopic masses to reference adduct m/z values for the calibrant list) — https://github.com/PNNL-m-q/mzapy
- **MZA** (HDF5-based file format from which measured calibrant m/z values are extracted via mzapy.MZA) — https://github.com/PNNL-m-q/mza
- **h5py** (Underlying HDF5 access layer for reading MZA files containing the raw calibrant spectra)

## Examples

```
from mzapy.isotopes import monoiso_mass, ms_adduct_mz; ref_mz = ms_adduct_mz(monoiso_mass(formula), adduct='[M+H]+')
```

## Evaluation signals

- Post-calibration ppm residuals for every calibrant fall within the tolerances enforced by the unittest.TestCase in the mzapy/test subpackage
- Each reference m/z from ms_adduct_mz is monotonically paired 1:1 with a measured m/z from the MZA file before fitting
- Pre- vs post-calibration ppm error comparison shows reduced magnitude of mass error across all calibrants in the residual table
- Monoisotopic masses match values recomputed from Unimod element masses via mzapy.isotopes.monoiso_mass

## Limitations

- Reference masses are only as accurate as the Unimod element masses and the assumed adduct assignment; a wrong adduct produces a systematically shifted reference m/z
- Calibration quality depends on having measured peaks in the raw MZA data for the selected calibrants; missing or mispicked calibrant peaks degrade the fit

## Evidence

- [other] Build the reference calibrant list by computing exact monoisotopic masses and adduct m/z values with mzapy.isotopes.monoiso_mass and mzapy.isotopes.ms_adduct_mz (element masses sourced from Unimod).: [evidence span withheld — anchor status: not_found]
- [abstract] mzapy contains supporting utilities enabling tasks including calibration: "In addition to raw data extraction, mzapy contains supporting utilities enabling tasks including calibration, signal processing, peak finding, and generating plots"
- [methods] Mass Calibration ... MassCalibration: [evidence span withheld — anchor status: not_found]
- [methods] monoiso_mass: [evidence span withheld — anchor status: not_found]
- [methods] ms_adduct_mz: [evidence span withheld — anchor status: not_found]
- [readme] MZA is a simple data structure based on the broadly supported HDF5 format: [evidence span withheld — anchor status: external_doc]
