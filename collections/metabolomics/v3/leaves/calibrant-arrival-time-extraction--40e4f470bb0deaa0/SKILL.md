---
name: calibrant-arrival-time-extraction
description: 'Use when a CCS calibration workflow needs calibrant ion drift/arrival
  times: raw IMS-MS data is in MZA (HDF5) format and per-calibrant ATDs and IonMobilityTime
  metadata must be pulled before fitting a calibration.'
when_to_use_negative:
- The data is not in MZA format or has no ion mobility dimension — mzapy targets 'complex
  data containing the ion mobility spectrometry dimension' and MZA files; plain LC-MS-only
  data needs no ATD extraction.
- Only mass calibration (no CCS) is needed — mass calibration uses mzapy.calibration.MassCalibration
  on m/z data and does not require arrival-time distributions.
- The file only contains pre-computed vendor CCS calibration coefficients (CCScalDT,
  CCScalSLIM, CCScalTIMS stored as HDF5 datasets during conversion) and no new calibrant
  ATDs need to be measured.
tools:
- name: mzapy
  role: Python package providing the MZA interface; mzapy.MZA.collect_atd_arrays_by_rt_mz
    performs the calibrant ATD extraction
  repo: https://github.com/PNNL-m-q/mzapy
- name: mza
  role: Standalone converter that produces the MZA/HDF5 files (from Agilent .d, Bruker
    .d, Thermo .raw, or mzML) that this extraction reads
  repo: https://github.com/PNNL-m-q/mza
- name: h5py
  role: HDF5 dependency underlying MZA file access by mzapy
- name: CCSCalibrationTW
  role: 'Downstream consumer: fits the TWIMS/SLIM CCS power-function calibration from
    the extracted calibrant drift times and reference CCS values'
  repo: https://github.com/PNNL-m-q/mzapy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:1686a5ed60f0030cf302d2273672bc80dcb0f2630e0df6f6db7241fd8540c9e7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - CCSCalibrationTW
  - Python
  - h5py
  - mza
  - mzapy
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# calibrant-arrival-time-extraction

## Summary

Extracts arrival-time distributions (ATDs) and ion-mobility metadata headers for calibrant ions from raw IMS-MS data stored in the MZA/HDF5 format using mzapy.MZA, providing the drift/arrival-time inputs required for downstream CCS calibration.

## When to use

Use when a CCS calibration workflow needs calibrant ion drift/arrival times: raw IMS-MS data is in MZA (HDF5) format and per-calibrant ATDs and IonMobilityTime metadata must be pulled before fitting a calibration.

## When NOT to use

- The data is not in MZA format or has no ion mobility dimension — mzapy targets 'complex data containing the ion mobility spectrometry dimension' and MZA files; plain LC-MS-only data needs no ATD extraction.
- Only mass calibration (no CCS) is needed — mass calibration uses mzapy.calibration.MassCalibration on m/z data and does not require arrival-time distributions.
- The file only contains pre-computed vendor CCS calibration coefficients (CCScalDT, CCScalSLIM, CCScalTIMS stored as HDF5 datasets during conversion) and no new calibrant ATDs need to be measured.

## Inputs

- MZA file (HDF5-based mass spectrometry data structure) containing an ion mobility dimension
- Calibrant list with reference m/z values, retention times, and literature CCS values (e.g. the mzapy repository's example calibrant dataset)
- Metadata table columns: Scan, RetentionTime, IonMobilityTime, IonMobilityBin, IonMobilityFrame

## Outputs

- Per-calibrant arrival-time distribution arrays (drift/arrival time vs intensity)
- Per-calibrant metadata headers (arrival time in milliseconds for DT and SLIM; 1/k0 in Vs/cm2 for TimsTOF)
- Calibrant drift-time/CCS pairs ready to feed mzapy.calibration.CCSCalibrationTW or MassCalibration fits

## How to apply

First convert vendor data (Agilent .d, Bruker .d, Thermo .raw, or mzML) to MZA with the mza.exe tool, then load the file with mzapy.MZA. Assemble the calibrant reference list (reference m/z values and literature CCS values, e.g. from the package's example calibrant dataset) and query the MZA file for each calibrant ion. Call mzapy.MZA.collect_atd_arrays_by_rt_mz to collect the arrival-time distribution arrays at the calibrant's retention time and m/z, and read the metadata table columns such as IonMobilityTime and IonMobilityBin for the relevant scans. The rationale is that MZA stores spectra as jagged 1D arrays (Arrays_mz/Arrays_intensity, or Arrays_mzbin/Full_mz_array for IM frames) indexed by Scan, so per-calibrant ATDs must be reassembled by query rather than read as a dense matrix. The extracted drift/arrival times plus reference CCS values are then handed to mzapy.calibration (e.g. CCSCalibrationTW for TWIMS/SLIM, or the single-field DTIMS method based on Anal. Chem. 2017, 89, 9048-9055).

## Related tools

- **mzapy** (Python package providing the MZA interface; mzapy.MZA.collect_atd_arrays_by_rt_mz performs the calibrant ATD extraction) — https://github.com/PNNL-m-q/mzapy
- **mza** (Standalone converter that produces the MZA/HDF5 files (from Agilent .d, Bruker .d, Thermo .raw, or mzML) that this extraction reads) — https://github.com/PNNL-m-q/mza
- **h5py** (HDF5 dependency underlying MZA file access by mzapy)
- **CCSCalibrationTW** (Downstream consumer: fits the TWIMS/SLIM CCS power-function calibration from the extracted calibrant drift times and reference CCS values) — https://github.com/PNNL-m-q/mzapy

## Examples

```
from mzapy import MZA; mza = MZA('calibrants.mza'); rt, mz, atd_arrays = mza.collect_atd_arrays_by_rt_mz(calibrant_rts, calibrant_mzs)
```

## Evaluation signals

- Extracted ATDs correspond to the calibrant scans: check metadata (Scan, RetentionTime, IonMobilityBin) matches the queried rt/mz for each calibrant ion.
- IonMobilityTime units are correct per instrument: milliseconds for DT and SLIM, 1/k0 (Vs/cm2) for TimsTOF — wrong units will silently corrupt the calibration fit.
- Downstream validation: the fitted calibration parameters must reproduce, within the tolerances encoded in the mzapy unittest.TestCase subclasses, the expected calibrated CCS values.
- Calibrant peak ATDs should be well-formed (peaks findable via mzapy.peaks.find_peaks_1d_gauss / find_peaks_1d_localmax); flat or multi-peak ATDs at the queried rt/mz indicate wrong query coordinates.
- Back-calculated CCS for each calibrant (via transform/calibrated_ccs) should agree with the literature CCS values used as references.

## Limitations

- Analysis of IMS data has been challenging and limited the full utility of these measurements; the extra IMS dimension requires upgrading existing computational pipelines, so ATD extraction assumes MZA/HDF5 storage.
- IonMobilityBin = 0 represents the total frame spectrum (summed across the IM dimension) — querying it yields no usable per-calibrant ATD.
- For files with too many spectra, arrays may be partitioned (MzaPath index), so array access must use the correct partition group (e.g. Arrays_intensity1/630).
- For Bruker TimsTOF the stored 'arrival time' is actually 1/k0 in Vs/cm2, not milliseconds, which limits direct reuse of these values with DT/SLIM-style calibration forms.

## Evidence

- [methods] ATD collection via mzapy.MZA: [evidence span withheld — anchor status: not_found]
- [abstract] mzapy scope covers IMS dimension: [evidence span withheld — anchor status: not_found]
- [readme] MZA stores jagged per-scan arrays: [evidence span withheld — anchor status: external_doc]
- [readme] IonMobilityTime units per instrument: [evidence span withheld — anchor status: external_doc]
- [readme] IonMobilityBin=0 is the frame sum: [evidence span withheld — anchor status: external_doc]
