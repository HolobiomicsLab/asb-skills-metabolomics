---
name: ddms2-spectrum-to-precursor-assignment
description: Use when centroided mzML files acquired under data-dependent ddMS2 must
  have their MS2 spectra linked to detected MS1 features before PFAS fragment matching,
  KMD or suspect-screening prioritization.
when_to_use_negative:
- The mzML data were not acquired with data-dependent acquisition (ddMS2) or contain
  profile-mode, non-centroided spectra — no reliable MS2-to-precursor linkage is possible
- You only have a custom/vendor feature list without the corresponding mzML files
  — MS2 data would be missing and data evaluation cannot proceed
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pyOpenMS
  role: Performs feature detection on the ddMS2 mzML data and supports the MS2 handling
    that links spectra to MS1 precursors
  repo: https://github.com/JonZwe/PFAScreen
- name: PFΔScreen
  role: Runs the FeatureFinding tab (feature finding, MS2 alignment at 5 mDa / 0.2
    min RT, blank correction) via the 'Run FeatureFinding' button
  repo: https://github.com/JonZwe/PFAScreen
- name: MSConvert
  role: Converts vendor raw data (e.g., Agilent .d) into the centroided ddMS2 mzML
    files consumed by this skill
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:87ecc39fdf2914406595f2b17d22ab3b9485debbd31fb044fe1d7cbef1d95155
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ddms2-spectrum-to-precursor-assignment

## Summary

Assigns each acquired ddMS2 spectrum in centroided mzML data to its MS1 precursor/feature during PFΔScreen's FeatureFinding step, aligning MS2 spectra with a 5 mDa mass tolerance and 0.2 min RT tolerance so that downstream PFAS fragment and diagnostic-fragment analysis has per-feature MS2 coverage.

## When to use

Use when centroided mzML files acquired under data-dependent ddMS2 must have their MS2 spectra linked to detected MS1 features before PFAS fragment matching, KMD or suspect-screening prioritization.

## When NOT to use

- The mzML data were not acquired with data-dependent acquisition (ddMS2) or contain profile-mode, non-centroided spectra — no reliable MS2-to-precursor linkage is possible
- You only have a custom/vendor feature list without the corresponding mzML files — MS2 data would be missing and data evaluation cannot proceed

## Inputs

- Centroided ddMS2 mzML sample file (e.g., converted from Agilent .d raw data with MSConvert)
- Optional blank mzML measured under the same ddMS2 acquisition
- Optional external feature list (external_feature_list.xlsx) with corresponding mzML files

## Outputs

- Feature list (feature_list.csv) with m/z, RT, intensity, and MS2 spectrum assignment per feature
- MS2-aligned spectrum set (5 mDa / 0.2 min RT tolerance) for downstream diagnostic-fragment and fragment-mass-difference analysis

## How to apply

After FeatureFinderMetabo detects MS1 features from the centroided ddMS2 mzML sample (10 ppm mass error, 2000-count MS1 intensity threshold, metabolites 5% RMS isotope model), assign to each MS1 precursor the MS2 spectra acquired by data-dependent acquisition. Align the MS2 spectra with a mass tolerance of 5 mDa and an RT tolerance of 0.2 min so that spectra shared across features are collapsed to unique spectra. The input mzML must be ddMS2 with centroided spectra, ideally one collision energy per precursor, as produced by MSConvert from vendor .d files. Report per-feature MS2 assignment alongside m/z, RT and intensity in the feature list consumed by PFASPrioritization, tracking how many acquired MS2 spectra map to detected features (in the paper: 1026 of 2450 MS2 spectra corresponded to features, 417 unique, ~11% MS2 coverage in the first iteration).

## Related tools

- **pyOpenMS** (Performs feature detection on the ddMS2 mzML data and supports the MS2 handling that links spectra to MS1 precursors) — https://github.com/JonZwe/PFAScreen
- **PFΔScreen** (Runs the FeatureFinding tab (feature finding, MS2 alignment at 5 mDa / 0.2 min RT, blank correction) via the 'Run FeatureFinding' button) — https://github.com/JonZwe/PFAScreen
- **MSConvert** (Converts vendor raw data (e.g., Agilent .d) into the centroided ddMS2 mzML files consumed by this skill)

## Examples

```
Run_PFAScreen.bat  # then in the GUI: Browse Sample.mzML (and optional Blank.mzML) → Run FeatureFinding to assign MS2 spectra to MS1 precursors
```

## Evaluation signals

- Every detected MS1 feature in feature_list.csv carries an MS2 assignment column, and rows without mzML-derived MS2 are flagged (data evaluation requires the corresponding mzML files)
- MS2 alignment metrics match the reported scale: of 2450 acquired MS2 spectra, 1026 corresponded to detected features and 417 unique spectra remained (~11% MS2 coverage in the first iteration)
- Aligned MS2 spectra use a mass tolerance of 5 mDa and an RT tolerance of 0.2 min; verify duplicate spectra across features collapse to unique spectra accordingly
- Input mzML passed an acquisition check: data-dependent ddMS2 with centroided spectra, ideally one collision energy per precursor
- MS2 assignments persist through blank correction (4209 features in soil M1 ESI− reduced to 3750) into the PFASPrioritization stage

## Limitations

- MS2 coverage is low (~11% of detected features had unique MS2 in the first iteration), so many features will lack fragment evidence for prioritization
- The skill depends on ddMS2 acquisition with centroided spectra; data evaluation fails if the corresponding mzML files are not provided alongside external feature lists
- PFΔScreen's reported runtimes (<1 min for ~4000 spectra) are given without hardware specifications, and no default parameter values for MS2 alignment are documented beyond the 5 mDa / 0.2 min tolerances

## Evidence

- [results] MS2 spectra were aligned with a mass tolerance of 5 mDa and an RT tolerance of 0.2 min: "MS2 spectra were aligned with a mass tolerance of 5 mDa and an RT tolerance of 0.2 min"
- [results] A total of 1026 out of 2450 acquired MS2 spectra corresponded to detected features, from which 417 unique spectra remained (~ 11% MS2 coverage in first iteration).: "A total of 1026 out of 2450 acquired MS2 spectra corresponded to detected features, from which 417 unique spectra remained (~ 11% MS2 coverage in first iteration)."
- [readme] Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor.: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor."
- [readme] Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing.: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
- [abstract] Feature detection from vendor-independent MS raw data (mzML, data-dependent acquisition) is performed via pyOpenMS: "Feature detection from vendor-independent MS raw data (mzML, data-dependent acquisition) is performed via pyOpenMS"
