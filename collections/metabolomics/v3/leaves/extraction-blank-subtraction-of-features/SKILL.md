---
name: extraction-blank-subtraction-of-features
description: Use when a centroided ddMS2 mzML sample (or custom feature list) is accompanied
  by a measured extraction blank mzML, and background signals from solvents, labware,
  or sample preparation must be removed before PFAS feature prioritization.
when_to_use_negative:
- The user supplies only a sample file with no extraction blank mzML — blank correction
  cannot be performed and features are passed unfiltered to prioritization
- The input is already a final prioritization results table rather than raw mzML or
  a pre-prioritization feature list
- The data are profile-mode or non-DDA spectra — PFΔScreen requires centroided ddMS2
  mzML input
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PFΔScreen
  role: Hosts the FeatureFinding tab where the blank-correction parameters (2 mDa,
    0.1 min RT, 5-fold abundance) are applied to sample vs. blank features
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Performs feature detection on both the sample and blank mzML via FeatureFinderMetabo,
    producing the feature lists that blank correction compares
  repo: https://github.com/JonZwe/PFAScreen
- name: MSConvert
  role: Converts vendor raw data (e.g., Agilent .d) into the centroided mzML format
    consumed by the blank-correction step
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:930e94d40a68af3c7d91cc329105fb5ba18e786507b0fdd492b5b7c33c0fa455
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
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# extraction-blank-subtraction-of-features

## Summary

Removes background features from an HRMS sample by matching detected features against an extraction blank: any feature present in both sample and blank (< 2 mDa mass deviation, < 0.1 min RT difference) is dropped unless it is at least fivefold more abundant in the sample, yielding a background-corrected feature list for downstream PFAS prioritization.

## When to use

Use when a centroided ddMS2 mzML sample (or custom feature list) is accompanied by a measured extraction blank mzML, and background signals from solvents, labware, or sample preparation must be removed before PFAS feature prioritization.

## When NOT to use

- The user supplies only a sample file with no extraction blank mzML — blank correction cannot be performed and features are passed unfiltered to prioritization
- The input is already a final prioritization results table rather than raw mzML or a pre-prioritization feature list
- The data are profile-mode or non-DDA spectra — PFΔScreen requires centroided ddMS2 mzML input

## Inputs

- Centroided ddMS2 mzML file of the sample (data-dependent acquisition, ideally one collision energy per precursor)
- Optional mzML file of the extraction blank control (Browse Blank.mzML)
- Alternatively, external feature tables (external_feature_list.xlsx / BlankFeatures.xlsx) plus the corresponding mzML files

## Outputs

- Blank-corrected feature list with m/z, RT, intensity, and per-feature MS2 assignment (feature_list.csv / external feature table)
- Feature count before and after blank correction for QC (e.g., 4209 → 3750 for soil M1 ESI−)

## How to apply

Run feature detection (pyOpenMS FeatureFinderMetabo, 10 ppm mass error, 2000-count MS1 intensity threshold, metabolites 5% RMS isotope model) independently on the sample mzML and the extraction blank mzML within PFΔScreen's FeatureFinding tab. For each sample feature, search the blank feature list for a match with mass deviation < 2 mDa and RT difference < 0.1 min; if matched, compare intensities and remove the feature unless the sample is at least fivefold more abundant than the blank. Execute the correction per soil together with its own extraction blank (e.g., soil M1 ESI− went from 4209 detected features to 3750 after blank correction). Write the blank-corrected feature list (m/z, RT, intensity, MS2 assignment per feature) to feature_list.csv as input for the PFASPrioritization stage. Note that features from custom/vendor feature lists can also be blank-corrected via the Browse BlankFeatures.xlsx route, but the corresponding mzML files must still be supplied so MS2 data remain available.

## Related tools

- **PFΔScreen** (Hosts the FeatureFinding tab where the blank-correction parameters (2 mDa, 0.1 min RT, 5-fold abundance) are applied to sample vs. blank features) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Performs feature detection on both the sample and blank mzML via FeatureFinderMetabo, producing the feature lists that blank correction compares) — https://github.com/JonZwe/PFAScreen
- **MSConvert** (Converts vendor raw data (e.g., Agilent .d) into the centroided mzML format consumed by the blank-correction step)

## Examples

```
python -c "from pyOpenMS import *; ff = MetaboliteFeatureFinding(); ff.run(sample_mzml, blank_mzml)"  # then in PFΔScreen GUI: Run FeatureFinding with sample + blank mzML
```

## Evaluation signals

- feature_list.csv exists with required m/z, RT, intensity, and MS2 columns
- Post-blank feature count is lower than the pre-blank count and on the reported scale (e.g., 4209 detected reduced to 3750 after blank correction in soil M1 ESI−)
- No retained feature matches a blank feature within < 2 mDa and < 0.1 min RT at less than fivefold sample abundance
- Each retained feature carries an MS2 assignment where the acquired ddMS2 spectra corresponded to detected features (1026 of 2450 MS2 spectra matched features in the study)
- Sample and blank mzML were both centroided DDA files with one collision energy per precursor, as required by PFΔScreen

## Limitations

- A measured extraction blank is required; without it background signals remain in the feature list
- Matching is limited to the strict tolerances (< 2 mDa, < 0.1 min RT, 5-fold ratio); features just outside these windows may survive as false positives, and PFΔScreen offers no quantitative benchmark against alternative data-reduction tools
- Custom feature lists used for blank correction still require the corresponding mzML files, otherwise MS2 data would be missing
- Default parameter values are not fixed — input parameters must be adjusted depending on the tested sample, instrument, and end-user needs

## Evidence

- [results] Blank correction removes features detected in both sample and extraction blank: "Features detected in both sample and extraction blank that deviated by < 2 mDa at a RT difference of < 0.1 min and were not at least fivefold more abundant in the sample were removed."
- [results] Feature counts before and after blank correction in soil M1: "Exemplified on soil M1, 4209 features were detected, which were reduced to 3750 features after blank correction in the ESI− mode."
- [results] Blank is loaded per sample and PFΔScreen run for background removal: "For each soil, PFΔScreen was executed individually together with the extraction blank to remove background signals"
- [readme] README: blank mzML is optional input to the FeatureFinding tab: "click the “Browse Sample.mzML” button and choose the mzML file of a sample and an optional mzML file of a blank control (Browse Blank.mzML)"
- [readme] README: custom feature lists still need corresponding mzML for MS2: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
