---
name: lc-hrms-feature-detection
description: Use when processing vendor-independent mzML files (data-dependent acquisition,
  centroided spectra) that need metabolomics-style feature detection, MS2-to-precursor
  assignment, and blank subtraction prior to feature prioritization.
when_to_use_negative:
- Input is already a curated feature table with no associated mzML — MS2 data would
  be missing and PFΔScreen data evaluation requires the corresponding mzML files
- Raw vendor data (e.g., Agilent .d) has not yet been converted to mzML via MSConvert
- Data are profile-mode or acquired without data-dependent MS2 — PFΔScreen expects
  ddMS2 with centroided spectra
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: pyOpenMS
  role: Performs feature detection (FeatureFinderMetabo) on mzML raw data within the
    PFΔScreen FeatureFinding tab
  repo: https://github.com/JonZwe/PFAScreen
- name: OpenMS
  role: Provides the metabolites (5% RMS) isotope model used to exclude features with
    unusual peak shapes of isotopic traces
- name: MSConvert
  role: Converts vendor raw MS data (.d files, Agilent) into the mzML format consumed
    by this skill
- name: PFΔScreen
  role: Hosts the FeatureFinding tab (GUI) that wraps feature finding, MS2 alignment,
    and blank correction
  repo: https://github.com/JonZwe/PFAScreen
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:5f21eebfae45774a5fa13f0b02e4e03449b4573b0869c602a650b3ee57fd9e19
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lc-hrms-feature-detection

## Summary

Detects LC-HRMS features from centroided ddMS2 mzML data using pyOpenMS FeatureFinderMetabo, then assigns MS2 spectra and performs blank correction, as implemented in PFΔScreen's FeatureFinding tab.

## When to use

Use when processing vendor-independent mzML files (data-dependent acquisition, centroided spectra) that need metabolomics-style feature detection, MS2-to-precursor assignment, and blank subtraction prior to feature prioritization.

## When NOT to use

- Input is already a curated feature table with no associated mzML — MS2 data would be missing and PFΔScreen data evaluation requires the corresponding mzML files
- Raw vendor data (e.g., Agilent .d) has not yet been converted to mzML via MSConvert
- Data are profile-mode or acquired without data-dependent MS2 — PFΔScreen expects ddMS2 with centroided spectra

## Inputs

- Sample mzML file (centroided spectra, data-dependent acquisition ddMS2)
- Optional blank (extraction control) mzML file measured under the same ddMS2 conditions
- Optional external feature list (external_feature_list.xlsx) with corresponding mzML files

## Outputs

- Blank-corrected feature list (feature_list.csv) with m/z, RT, intensity, and MS2 assignment per feature

## How to apply

Load the centroided ddMS2 mzML sample file (and optional extraction-blank mzML) with pyOpenMS and run FeatureFinderMetabo with a mass error of 10 ppm, an MS1 intensity threshold of 2000 counts, and the metabolites (5% RMS) isotope model from OpenMS. Filter detected features to those with FWHM above 1 s and below 1 min and at least two isotopic traces, then assign each MS1 precursor the single highest-intensity MS2 spectrum from the data-dependent acquisition. Align MS2 spectra with a mass tolerance of 5 mDa and an RT tolerance of 0.2 min. Apply blank correction: remove features detected in both sample and extraction blank that deviate by < 2 mDa at an RT difference of < 0.1 min unless they are at least fivefold more abundant in the sample. Write the blank-corrected feature list (m/z, RT, intensity, MS2 assignment) to feature_list.csv for downstream PFASPrioritization; alternatively, custom feature lists (external_feature_list.xlsx) can be supplied with the corresponding mzML files so MS2 data are still available.

## Related tools

- **pyOpenMS** (Performs feature detection (FeatureFinderMetabo) on mzML raw data within the PFΔScreen FeatureFinding tab) — https://github.com/JonZwe/PFAScreen
- **OpenMS** (Provides the metabolites (5% RMS) isotope model used to exclude features with unusual peak shapes of isotopic traces)
- **MSConvert** (Converts vendor raw MS data (.d files, Agilent) into the mzML format consumed by this skill)
- **PFΔScreen** (Hosts the FeatureFinding tab (GUI) that wraps feature finding, MS2 alignment, and blank correction) — https://github.com/JonZwe/PFAScreen

## Examples

```
Double click Run_PFAScreen.bat, click "Browse Sample.mzML" (and "Browse Blank.mzML"), set FeatureFinding parameters, then press "Run FeatureFinding"
```

## Evaluation signals

- feature_list.csv exists and contains the required m/z, RT, intensity, and MS2 columns
- Feature counts are on the reported scale (e.g., 4209 features detected in soil M1 ESI− reduced to 3750 after blank correction)
- Retained features satisfy the FWHM (1 s to 1 min) and at-least-two-isotopic-traces constraints
- No retained feature co-occurs in blank within 2 mDa / RT < 0.1 min unless ≥5-fold more abundant in sample
- MS2 alignment used 5 mDa mass tolerance and 0.2 min RT tolerance; check MS2 assignments cover detected features

## Limitations

- Sample and blank should be measured under ddMS2 with centroided spectra, ideally with one collision energy per precursor; other acquisition modes are not supported
- PFΔScreen version number is not stated and no default or recommended input parameter values are provided — parameters must be adjusted per sample and instrument
- No changelog is available for the PFAScreen repository; Python 3.9 is required for automated installation

## Evidence

- [results] Feature detection with mass error 10 ppm, MS1 intensity threshold 2000 counts, OpenMS isotope model: "The mass error for feature detection was set to 10 ppm, the MS1 intensity threshold was set to 2000 counts and the metabolites (5% RMS) isotope model from OpenMS was used"
- [results] Blank correction rule (2 mDa, RT < 0.1 min, fivefold): "Features detected in both sample and extraction blank that deviated by < 2 mDa at a RT difference of < 0.1 min and were not at least fivefold more abundant in the sample were removed."
- [results] FWHM and isotopic trace filter: "Peaks reported after feature detection have to have a full width at half maximum (FWHM) above 1 s and below 1 min, and at least two isotopic traces."
- [results] Feature counts on soil M1 ESI−: "Exemplified on soil M1, 4209 features were detected, which were reduced to 3750 features after blank correction in the ESI− mode."
- [readme] ddMS2 centroided input requirement: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor."
