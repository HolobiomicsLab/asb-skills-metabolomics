---
name: calibration-fit-statistics-validation
description: Use when calibrating CCS from TWIMS or SLIM arrival times of calibrant
  ions in MZA/HDF5 data, and the fitted power-function parameters (slope/intercept/fit
  statistics) must be verified against expected values before being trusted downstream.
when_to_use_negative:
- The data has no ion mobility dimension (no arrival-time distributions) — mzapy's
  calibration utilities target IMS-MS data; plain LC-MS quantitation or mass-only
  workflows should use mass calibration (MassCalibration) instead
- The instrument is a Bruker TimsTOF where the stored quantity is 1/k0 (Vs/cm2) rather
  than arrival time in ms — vendor CCScalTIMS coefficients apply, not the TWIMS/SLIM
  power-function calibration
- The task is only raw data extraction or peak finding (find_peaks_1d_localmax, find_peaks_1d_gauss)
  with no calibration step required
tools:
- name: mzapy
  role: Provides CCSCalibrationTW (TWIMS/SLIM CCS calibration) with the inherited
    _CalibrationBase.fit method, transform/calibrated_ccs back-calculation, mzapy.view.plot_tw_ccs_calibration
    visualization, and mzapy.test unittest tolerances used for validation
  repo: https://github.com/PNNL-m-q/mzapy
- name: mzapy.MZA
  role: Reads the MZA/HDF5 calibrant dataset and extracts calibrant arrival-time distributions
    and metadata headers via collect_atd_arrays_by_rt_mz
  repo: https://github.com/PNNL-m-q/mzapy
- name: h5py
  role: Underlying HDF5 access dependency for reading MZA calibrant data
  repo: https://github.com/PNNL-m-q/mzapy
- name: mza
  role: Command-line converter that produces the MZA files and embeds vendor CCS calibration
    coefficients (CCScalSLIM, CCScalDT) as HDF5 datasets when detected
  repo: https://github.com/PNNL-m-q/mza
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:79f627476cf5ac382f0793ad2cbb5ce82134c9b8f9dcf778a3424c21e4aad241
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - h5py
  - mza
  - mzapy
  - mzapy.MZA
  - mzapy.calibration.CCSCalibrationTW
  - mzapy.calibration.tw_ccs_calibration_from_params
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# calibration-fit-statistics-validation

## Summary

Fits and validates collision cross section (CCS) calibration functions for TWIMS/SLIM ion mobility data using mzapy.calibration, checking the fitted power-function parameters and back-calculated calibrant CCS values against the tolerances asserted in mzapy's unit tests.

## When to use

Use when calibrating CCS from TWIMS or SLIM arrival times of calibrant ions in MZA/HDF5 data, and the fitted power-function parameters (slope/intercept/fit statistics) must be verified against expected values before being trusted downstream.

## When NOT to use

- The data has no ion mobility dimension (no arrival-time distributions) — mzapy's calibration utilities target IMS-MS data; plain LC-MS quantitation or mass-only workflows should use mass calibration (MassCalibration) instead
- The instrument is a Bruker TimsTOF where the stored quantity is 1/k0 (Vs/cm2) rather than arrival time in ms — vendor CCScalTIMS coefficients apply, not the TWIMS/SLIM power-function calibration
- The task is only raw data extraction or peak finding (find_peaks_1d_localmax, find_peaks_1d_gauss) with no calibration step required

## Inputs

- Calibrant dataset in MZA/HDF5 format (e.g. the example calibrant dataset shipped in the mzapy repository)
- Calibrant reference (literature) masses and literature CCS values
- Calibrant arrival-time distributions and relevant metadata headers extracted from the raw MZA file (collect_atd_arrays_by_rt_mz)

## Outputs

- Fitted power-function calibration parameters (slope/intercept) from CCSCalibrationTW._CalibrationBase.fit
- Fit statistics for the calibration curve
- Back-calculated CCS values for each calibrant (transform/calibrated_ccs)
- Calibration plot produced by mzapy.view.plot_tw_ccs_calibration

## How to apply

Load the package's example calibrant dataset (MZA/HDF5 format) and assemble calibrant reference masses with literature CCS values. Extract each calibrant ion's arrival-time distribution and metadata headers from the raw file via mzapy.MZA (e.g. collect_atd_arrays_by_rt_mz). Instantiate mzapy.calibration.CCSCalibrationTW — the TWIMS/SLIM implementation combining the Nat. Protoc. 2008 dt/CCS corrections with the Anal. Chem. 2016 power-function form — and call the inherited _CalibrationBase.fit method on the calibrant drift times and reference CCS values. Back-calculate CCS for every calibrant with transform/calibrated_ccs, record the fitted power-function parameters and fit statistics, and visualize with mzapy.view.plot_tw_ccs_calibration. Validation is decisive: the fitted calibration (slope/intercept/fit statistics) must reproduce, within the tolerances encoded in mzapy.test unittest.TestCase subclasses, the expected calibrated CCS values; any deviation beyond those test tolerances fails.

## Related tools

- **mzapy** (Provides CCSCalibrationTW (TWIMS/SLIM CCS calibration) with the inherited _CalibrationBase.fit method, transform/calibrated_ccs back-calculation, mzapy.view.plot_tw_ccs_calibration visualization, and mzapy.test unittest tolerances used for validation) — https://github.com/PNNL-m-q/mzapy
- **mzapy.MZA** (Reads the MZA/HDF5 calibrant dataset and extracts calibrant arrival-time distributions and metadata headers via collect_atd_arrays_by_rt_mz) — https://github.com/PNNL-m-q/mzapy
- **h5py** (Underlying HDF5 access dependency for reading MZA calibrant data) — https://github.com/PNNL-m-q/mzapy
- **mza** (Command-line converter that produces the MZA files and embeds vendor CCS calibration coefficients (CCScalSLIM, CCScalDT) as HDF5 datasets when detected) — https://github.com/PNNL-m-q/mza

## Examples

```
from mzapy.calibration import CCSCalibrationTW; cal = CCSCalibrationTW(); cal.fit(tds, ccss); ccs_fit = cal.transform(tds)
```

## Evaluation signals

- Fitted power-function parameters must reproduce the expected calibrated CCS values within the tolerances encoded in mzapy.test unittest.TestCase subclasses; deviation beyond those tolerances fails the check
- Back-calculated CCS from transform/calibrated_ccs for each calibrant must match the assembled literature CCS reference values within the asserted test tolerances
- Slope/intercept and fit statistics of the fitted calibration function must agree with the values asserted in mzapy's unit tests and with the reported TWIMS/SLIM calibration literature
- The mzapy.view.plot_tw_ccs_calibration plot should show calibrant points and the fitted curve consistent with the recorded fit statistics

## Limitations

- CCSCalibrationTW addresses TWIMS/SLIM CCS calibration; single-field DTIMS CCS calibration is a separate method based on Anal. Chem. 2017, 89, 9048-9055
- Vendor CCS calibration coefficients (CCScalDT = [Tfix, Beta], CCScalSLIM = [C0, C1, C2, C3]) are only included in the MZA file if detected during conversion
- Ion mobility arrival time units differ by vendor: milliseconds for DT and SLIM, but 1/k0 in Vs/cm2 for TimsTOF, so arrival-time inputs must be interpreted per instrument type

## Evidence

- [abstract] mzapy supports calibration utilities: "In addition to raw data extraction, mzapy contains supporting utilities enabling tasks including calibration, signal processing, peak finding, and generating plots"
- [methods] DTIMS CCS calibration method reference: "CCS for single field DTIMS, method based on: - *Anal. Chem. 2017, 89, 9048-9055*"
- [abstract] MZA file format basis: "MZA, a new and simple mass spectrometry data structure based on the broadly supported HDF5 format"
- [readme] SLIM CCS calibration coefficients embedded by converter: [evidence span withheld — anchor status: external_doc]
- [readme] DT CCS calibration coefficients embedded by converter: [evidence span withheld — anchor status: external_doc]
- [methods] ATD extraction function: [evidence span withheld — anchor status: not_found]
