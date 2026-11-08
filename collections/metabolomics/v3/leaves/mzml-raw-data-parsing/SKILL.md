---
name: mzml-raw-data-parsing
description: Use when raw LC/GC-HRMS data must be read from vendor-independent mzML
  files (centroided, DDA/ddMS2) into Python via pyOpenMS as the entry point for feature
  finding or feature prioritization workflows.
when_to_use_negative:
- Input is already a processed feature list without the corresponding mzML files —
  MS2 data would be missing, and data evaluation only works when the mzML files are
  also provided
- Raw data are profile-mode or were acquired without data-dependent ddMS2 acquisition
  — PFΔScreen expects centroided spectra with DDA
- Data are in a vendor-specific raw format (.d, .raw, etc.) that has not first been
  converted to mzML via MSConvert
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pyOpenMS
  role: Python interface to the C++ OpenMS library used to parse mzML raw data and
    perform feature detection in PFΔScreen
  repo: https://github.com/JonZwe/PFAScreen
- name: MSConvert
  role: Converts vendor raw MS files (e.g., Agilent .d) into the mzML format consumed
    by this parsing step
- name: PFΔScreen
  role: GUI tool whose FeatureFinding tab consumes the parsed mzML input via pyOpenMS
    for feature detection, MS2 alignment, and blank correction
  repo: https://github.com/JonZwe/PFAScreen
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:06fb2f3abbd98e26bd2ba8d929f085c75040fdb4330bc0bdc229f98a13b9cf59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - OpenMS
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mzml-raw-data-parsing

## Summary

Loads and parses vendor-independent, centroided mzML raw MS data (data-dependent ddMS2 acquisition) into pyOpenMS so that downstream PFΔScreen feature detection, MS2-to-precursor assignment, and blank correction can operate on MS1 and MS2 spectra from any instrument vendor.

## When to use

Use when raw LC/GC-HRMS data must be read from vendor-independent mzML files (centroided, DDA/ddMS2) into Python via pyOpenMS as the entry point for feature finding or feature prioritization workflows.

## When NOT to use

- Input is already a processed feature list without the corresponding mzML files — MS2 data would be missing, and data evaluation only works when the mzML files are also provided
- Raw data are profile-mode or were acquired without data-dependent ddMS2 acquisition — PFΔScreen expects centroided spectra with DDA
- Data are in a vendor-specific raw format (.d, .raw, etc.) that has not first been converted to mzML via MSConvert

## Inputs

- Centroided mzML file of a sample measured with data-dependent ddMS2 acquisition (converted from vendor raw data, e.g., Agilent .d, via MSConvert)
- Optional centroided mzML file of an extraction blank control
- Optional external feature list (external_feature_list.xlsx) with its corresponding mzML files

## Outputs

- In-memory pyOpenMS representation of MS1 and MS2 spectra available to the FeatureFinding tab
- Parsed MS2 spectra enabling precursor assignment and downstream PFΔScreen prioritization (feature_list.csv with m/z, RT, intensity, MS2 assignment)

## How to apply

Convert raw instrument files (e.g., Agilent .d) to mzML with MSConvert, ensuring data-dependent acquisition (ddMS2) with centroided spectra and ideally one collision energy per precursor. Load the sample mzML (and optional blank mzML) into Python with pyOpenMS, the Python interface to the C++ OpenMS library, which provides both the MS1 peak data needed for feature detection (FeatureFinderMetabo with 10 ppm mass error, 2000-count MS1 intensity threshold, metabolites (5% RMS) isotope model) and the MS2 spectra needed for precursor assignment and blank correction. If feature finding is done elsewhere, custom feature lists (external_feature_list.xlsx) can be supplied, but the corresponding mzML files must still be parsed since MS2 data would otherwise be missing. Validate parsing by confirming the file structure supports the expected outputs (e.g., 4209 features detected in soil M1 ESI− from ~2450 acquired MS2 spectra) before passing the feature list onward.

## Related tools

- **pyOpenMS** (Python interface to the C++ OpenMS library used to parse mzML raw data and perform feature detection in PFΔScreen) — https://github.com/JonZwe/PFAScreen
- **MSConvert** (Converts vendor raw MS files (e.g., Agilent .d) into the mzML format consumed by this parsing step)
- **PFΔScreen** (GUI tool whose FeatureFinding tab consumes the parsed mzML input via pyOpenMS for feature detection, MS2 alignment, and blank correction) — https://github.com/JonZwe/PFAScreen

## Examples

```
from pyOpenMS import *; exp = MSExperiment(); MzMLFile().load('sample.mzML', exp)
```

## Evaluation signals

- mzML input contains centroided, data-dependent ddMS2 spectra (ideally one collision energy per precursor) as required by PFΔScreen
- pyOpenMS successfully loads the sample and blank mzML files and exposes both MS1 peaks (for feature detection with 10 ppm mass error, 2000-count MS1 threshold) and MS2 spectra (for alignment)
- Downstream feature counts land on the reported scale, e.g., 4209 features detected in soil M1 ESI− reduced to 3750 after blank correction
- MS2 coverage is plausible relative to acquisition, e.g., 1026 of 2450 acquired MS2 spectra corresponding to detected features
- If using external feature lists, the corresponding mzML files are present — otherwise MS2 data would be missing and evaluation cannot proceed

## Limitations

- PFΔScreen requires ddMS2 centroided mzML input; profile data or non-DDA acquisitions are not supported
- Custom feature lists can substitute for pyOpenMS feature detection, but the corresponding mzML files must still be provided or MS2 data would be missing
- No specific mass spectrometer vendors or instrument models are named in the article to substantiate the vendor-independence claim
- No default or recommended input parameter values are provided; parameters must be adjusted per sample and instrument

## Evidence

- [abstract] Feature detection from vendor-independent MS raw data (mzML, data-dependent acquisition) is performed via pyOpenMS (or custom feature lists): "Feature detection from vendor-independent MS raw data (mzML, data-dependent acquisition) is performed via pyOpenMS (or custom feature lists)"
- [readme] Sample and blank mzML requirements for PFΔScreen input: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor."
- [results] MSConvert conversion of raw Agilent .d files to mzML: "the raw MS data (.d files, Agilent) were converted into mzML with MSConvert"
- [results] Feature detection parameters applied to parsed mzML data: "The mass error for feature detection was set to 10 ppm, the MS1 intensity threshold was set to 2000 counts"
- [readme] Custom feature lists still require the mzML files for MS2 data: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
