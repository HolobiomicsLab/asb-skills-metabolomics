---
name: calibration-power-function-fitting
description: Use when drift/arrival-time distributions have been extracted from TWIMS
  or SLIM IMS-MS data (e.g. MZA/HDF5 files) and calibrant ions with literature CCS
  values are available, and the goal is to fit or apply a CCS calibration.
when_to_use_negative:
- Data are from a Bruker TimsTOF, whose CCScalTIMS = [C0..C9] coefficients encode
  a different 10-parameter model rather than the TWIMS/SLIM power-function form
- The task is single-field DTIMS CCS calibration, which in mzapy uses a separate method
  based on Anal. Chem. 2017, 89, 9048-9055
- The task is m/z (mass) calibration, handled by mzapy's MassCalibration utility instead
tools:
- name: mzapy
  role: Python package whose mzapy.calibration module provides CCSCalibrationTW for
    TWIMS/SLIM CCS calibration and tw_ccs_calibration_from_params for rebuilding a
    calibration from parameters
  repo: https://github.com/PNNL-m-q/mzapy
- name: mzapy.MZA
  role: Extracts calibrant arrival-time distributions and metadata headers (e.g. collect_atd_arrays_by_rt_mz)
    from the raw MZA/HDF5 file as calibration inputs
  repo: https://github.com/PNNL-m-q/mzapy
- name: h5py
  role: Underlying HDF5 access layer for reading the MZA calibrant dataset
- name: matplotlib
  role: Backing library for visualizing the fitted calibration via mzapy.view.plot_tw_ccs_calibration
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:a5dafba9c19e71dfc1ff6fbe7a806eff4caf4a72696abf84195ea3f868aa8813
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - h5py
  - matplotlib
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

# calibration-power-function-fitting

## Summary

Fit a power-function CCS calibration for TWIMS/SLIM ion mobility data using mzapy.calibration.CCSCalibrationTW, converting calibrant arrival times into collision cross sections validated against the mzapy unit-test tolerances and the underlying TWIMS/SLIM calibration literature.

## When to use

Use when drift/arrival-time distributions have been extracted from TWIMS or SLIM IMS-MS data (e.g. MZA/HDF5 files) and calibrant ions with literature CCS values are available, and the goal is to fit or apply a CCS calibration.

## When NOT to use

- Data are from a Bruker TimsTOF, whose CCScalTIMS = [C0..C9] coefficients encode a different 10-parameter model rather than the TWIMS/SLIM power-function form
- The task is single-field DTIMS CCS calibration, which in mzapy uses a separate method based on Anal. Chem. 2017, 89, 9048-9055
- The task is m/z (mass) calibration, handled by mzapy's MassCalibration utility instead

## Inputs

- MZA/HDF5 calibrant dataset from the mzapy repository
- Calibrant reference masses and literature CCS values
- Calibrant arrival-time distributions and metadata headers extracted from the raw file (IonMobilityTime in ms for DT and SLIM)

## Outputs

- Fitted power-function calibration parameters and fit statistics
- Back-calculated (calibrated) CCS values for each calibrant ion
- Calibration plot from mzapy.view.plot_tw_ccs_calibration

## How to apply

Load the calibrant dataset in MZA/HDF5 format and assemble calibrant reference masses and literature CCS values. Extract calibrant arrival-time distributions and metadata headers with mzapy.MZA (e.g. collect_atd_arrays_by_rt_mz) so a drift/arrival time is obtained for each calibrant ion. Instantiate mzapy.calibration.CCSCalibrationTW, which implements the Nat. Protoc. 2008 dt/CCS corrections in the Anal. Chem. 2016 power-function form, and call the inherited _CalibrationBase.fit method on the calibrant drift times and reference CCS values. Back-calculate CCS for each calibrant via transform/calibrated_ccs and record the fitted power-function parameters and fit statistics; visualize with mzapy.view.plot_tw_ccs_calibration. As a decision point, choose CCSCalibrationTW for TWIMS/SLIM data rather than the single-field DTIMS calibration, which follows the distinct method of Anal. Chem. 2017, 89, 9048-9055.

## Related tools

- **mzapy** (Python package whose mzapy.calibration module provides CCSCalibrationTW for TWIMS/SLIM CCS calibration and tw_ccs_calibration_from_params for rebuilding a calibration from parameters) — https://github.com/PNNL-m-q/mzapy
- **mzapy.MZA** (Extracts calibrant arrival-time distributions and metadata headers (e.g. collect_atd_arrays_by_rt_mz) from the raw MZA/HDF5 file as calibration inputs) — https://github.com/PNNL-m-q/mzapy
- **h5py** (Underlying HDF5 access layer for reading the MZA calibrant dataset)
- **matplotlib** (Backing library for visualizing the fitted calibration via mzapy.view.plot_tw_ccs_calibration)

## Examples

```
from mzapy.calibration import CCSCalibrationTW; cal = CCSCalibrationTW(dt, ccs); cal.fit(dt, ccs)
```

## Evaluation signals

- Fitted power-function parameters (slope/intercept and fit statistics) must match the values asserted in the mzapy unittest.TestCase subclasses within the tolerances encoded in mzapy.test
- Back-calculated calibrated_ccs values for each calibrant ion must agree with literature CCS references within those test tolerances
- Visual agreement of the calibration curve when rendered with mzapy.view.plot_tw_ccs_calibration
- For SLIM data converted to MZA, consistency of the fitted coefficients with the CCScalSLIM = [C0, C1, C2, C3] parameterization stored as an HDF5 dataset during conversion

## Limitations

- The power-function form is specific to TWIMS/SLIM CCS calibration; single-field DTIMS uses a different method (Anal. Chem. 2017, 89, 9048-9055) and TimsTOF data uses CCScalTIMS coefficients
- Calibration quality depends on extracting clean arrival-time distributions for each calibrant ion; IonMobilityTime units differ by platform (ms for DT and SLIM, 1/k0 in Vs/cm2 for TimsTOF)

## Evidence

- [methods] CCS calibration with CCSCalibrationTW and the Anal. Chem. 2016 power-function form: "CCS for single field DTIMS, method based on: - *Anal. Chem. 2017, 89, 9048-9055*"
- [abstract] mzapy provides calibration as a supporting utility beyond raw data extraction: "In addition to raw data extraction, mzapy contains supporting utilities enabling tasks including calibration, signal processing, peak finding, and generating plots"
- [readme] SLIM CCS calibration coefficients are stored as an HDF5 dataset during MZA conversion: [evidence span withheld — anchor status: external_doc]
- [methods] Arrival-time distributions are collected from MZA files with collect_atd_arrays_by_rt_mz: [evidence span withheld — anchor status: not_found]
