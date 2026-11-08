---
name: ppm-mass-accuracy-residual-analysis
description: Use when raw MZA spectra contain known calibrant peaks and you need to
  quantify mass error in ppm, fit a calibration against a reference list, and report
  residual tables for accuracy assessment.
when_to_use_negative:
- When the goal is extracting spectra/XICs/ATDs from MZA files rather than calibrating
  m/z values — use mzapy's raw extraction and collect_xic_arrays_by_mz utilities instead
- When calibrating collision cross section rather than m/z — mzapy's CCS calibration
  for single-field DTIMS is a separate routine
- When raw data have not been converted to the MZA format; vendor files (.d, .raw,
  mzML) must first be converted with the mza executable
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzapy
  role: Provides MassCalibration.fit/calibrated_mass/transform for fitting and applying
    the mass calibration, plus isotopes.monoiso_mass and isotopes.ms_adduct_mz to
    compute reference calibrant m/z values
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: HDF5-based file format holding the measured m/z and intensity arrays extracted
    prior to calibration
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: Dependency used by mzapy to read the MZA HDF5 datasets (Arrays_mz, Arrays_intensity)
    during raw data extraction
- name: Python
  role: Implementation language in which the calibration workflow is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:7e84bb542be0bcb1e326e24854943e1869f2176457a1d017580148a9300d1b66
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - Python
  - h5py
  - mzapy
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# ppm-mass-accuracy-residual-analysis

## Summary

Fits a mass calibration model (mzapy.calibration.MassCalibration) that maps measured m/z values onto reference calibrant masses, then quantifies mass accuracy as ppm residuals before and after calibration. It leverages mzapy's supporting utilities beyond raw data extraction to validate instrument mass accuracy.

## When to use

Use when raw MZA spectra contain known calibrant peaks and you need to quantify mass error in ppm, fit a calibration against a reference list, and report residual tables for accuracy assessment.

## When NOT to use

- When the goal is extracting spectra/XICs/ATDs from MZA files rather than calibrating m/z values — use mzapy's raw extraction and collect_xic_arrays_by_mz utilities instead
- When calibrating collision cross section rather than m/z — mzapy's CCS calibration for single-field DTIMS is a separate routine
- When raw data have not been converted to the MZA format; vendor files (.d, .raw, mzML) must first be converted with the mza executable

## Inputs

- Raw MZA file (HDF5-based) containing measured m/z values for calibrant spectra, read via mzapy.MZA and h5py
- List of calibrant formulas/adducts used to compute reference exact masses (mzapy.isotopes.monoiso_mass, ms_adduct_mz)

## Outputs

- Fitted MassCalibration model object (mzapy.calibration.MassCalibration)
- Calibrated m/z values for each calibrant (via calibrated_mass/transform)
- Residual table of pre- and post-calibration ppm mass errors (mass_calibration_residuals.csv)

## How to apply

1. Build the reference calibrant list by computing exact monoisotopic masses with mzapy.isotopes.monoiso_mass and converting to adduct m/z with mzapy.isotopes.ms_adduct_mz (element masses sourced from Unimod). 2. Extract the measured calibrant m/z values from the raw MZA file via mzapy.MZA and pair them with the corresponding reference masses. 3. Fit a mass calibration model using mzapy.calibration.MassCalibration.fit (inherited from mzapy.calibration._CalibrationBase), then apply it with MassCalibration.calibrated_mass/transform to obtain calibrated m/z values for every calibrant. 4. Compute mass errors in ppm before and after calibration for each calibrant and write the residual table to mass_calibration_residuals.csv. Validation requires post-calibration ppm residuals to fall within tolerances enforced by the unittest.TestCase in the mzapy/test subpackage.

## Related tools

- **mzapy** (Provides MassCalibration.fit/calibrated_mass/transform for fitting and applying the mass calibration, plus isotopes.monoiso_mass and isotopes.ms_adduct_mz to compute reference calibrant m/z values) — https://github.com/PNNL-m-q/mzapy
- **MZA** (HDF5-based file format holding the measured m/z and intensity arrays extracted prior to calibration) — https://github.com/PNNL-m-q/mza
- **h5py** (Dependency used by mzapy to read the MZA HDF5 datasets (Arrays_mz, Arrays_intensity) during raw data extraction)
- **Python** (Implementation language in which the calibration workflow is executed)

## Examples

```
from mzapy import MZA; from mzapy.calibration import MassCalibration; cal = MassCalibration(); cal.fit(measured_mz, ref_mz); cal_mz = cal.calibrated_mass(measured_mz)
```

## Evaluation signals

- Post-calibration ppm residuals for every calibrant fall within the tolerances enforced by the corresponding unittest.TestCase in the mzapy/test subpackage
- Residual table mass_calibration_residuals.csv is produced with pre- and post-calibration ppm errors for each calibrant
- Post-calibration ppm errors are smaller in magnitude than pre-calibration ppm errors for the same calibrants
- Reference calibrant m/z values match independently computed exact monoisotopic masses + adduct m/z from Unimod-sourced element masses

## Limitations

- Calibration quality depends on correctly matching measured peaks to reference calibrants; element/adduct masses derive from Unimod, so nonstandard adducts may not be covered
- The article notes mzapy is structured to support future extension, and no changelog was found, so calibration model behavior may evolve between versions
- Raw vendor files must first be converted to MZA with the mza converter before this skill can be applied

## Evidence

- [readme] MZA structure description from README: [evidence span withheld — anchor status: external_doc]
