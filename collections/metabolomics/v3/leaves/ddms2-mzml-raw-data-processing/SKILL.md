---
name: ddms2-mzml-raw-data-processing
description: Use when vendor LC/GC-HRMS raw data (e.g., Agilent .d) must be converted
  to centroided ddMS2 mzML and feature-detected with pyOpenMS (10 ppm, 2000 counts)
  plus MS2 alignment before PFAS feature prioritization.
when_to_use_negative:
- The corresponding mzML files are missing even though an external feature table (SampleFeatures.xlsx)
  exists — PFΔScreen data evaluation then lacks all MS2 data and cannot run.
- Raw data are profile spectra or DDA runs with multiple collision energies — the
  tool expects ddMS2 with centroided spectra, ideally one collision energy per precursor.
- The goal is routine targeted quantification of <50 known analytes rather than feature
  detection/alignment for prioritizing unknown PFAS in non-target HRMS data.
tools:
- name: PFΔScreen (PFAScreen)
  role: GUI/console that loads the ddMS2 mzML sample and optional blank ('Browse Sample.mzML'
    / 'Browse Blank.mzML') and executes 'Run FeatureFinding' and 'Run PFASPrioritization'
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Python interface to OpenMS performing feature detection in the mzML raw data
    (mass error 10 ppm, MS1 intensity threshold 2000 counts, FWHM and isotope-trace
    filtering)
- name: MSConvert
  role: Converts vendor raw MS data (Agilent .d) into vendor-independent mzML consumed
    by PFΔScreen
- name: OpenMS
  role: Supplies the 'metabolites (5% RMS)' isotope model used to exclude features
    with unusual peak shapes of isotopic traces
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:8ca05fbe5039bf0ac8e76ae9c158ac165f0c9d691f222a683fd048b3475988aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - OpenMS
  - PFΔScreen
  - PFΔScreen (PFAScreen)
  - Python
  - pyOpenMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# ddms2-mzml-raw-data-processing

## Summary

Converts vendor LC/GC-HRMS raw files (e.g., Agilent .d) to centroided ddMS2 mzML via MSConvert and runs pyOpenMS-based feature finding plus MS2 spectral alignment inside PFΔScreen, producing the blank-corrected feature and MS2 tables that feed PFAS prioritization (MD/C-m/C, KMD analysis, diagnostic fragments, suspect screening).

## When to use

Use when vendor LC/GC-HRMS raw data (e.g., Agilent .d) must be converted to centroided ddMS2 mzML and feature-detected with pyOpenMS (10 ppm, 2000 counts) plus MS2 alignment before PFAS feature prioritization.

## When NOT to use

- The corresponding mzML files are missing even though an external feature table (SampleFeatures.xlsx) exists — PFΔScreen data evaluation then lacks all MS2 data and cannot run.
- Raw data are profile spectra or DDA runs with multiple collision energies — the tool expects ddMS2 with centroided spectra, ideally one collision energy per precursor.
- The goal is routine targeted quantification of <50 known analytes rather than feature detection/alignment for prioritizing unknown PFAS in non-target HRMS data.

## Inputs

- Vendor LC/GC-HRMS raw data (e.g., Agilent .d) acquired in ddMS2/DDA mode
- Centroided mzML sample file converted with MSConvert (paper benchmark: ~4000 spectra per sample)
- Optional blank/extraction-blank mzML measured under the same ddMS2 conditions
- Optional external feature list (external_feature_list.xlsx / SampleFeatures.xlsx) with its corresponding mzML files

## Outputs

- pyOpenMS feature list (m/z, RT, FWHM, isotopic traces) filtered to FWHM >1 s and <1 min with >=2 isotopic traces
- Blank-corrected feature table (e.g., 4209 -> 3750 features for soil M1 ESI−)
- MS2 spectra aligned to detected features (5 mDa mass tolerance, 0.2 min RT tolerance) feeding PFASPrioritization (Excel results table and interactive HTML plots)

## How to apply

Convert vendor raw files (e.g., Agilent .d) to vendor-independent mzML with MSConvert; PFΔScreen expects ddMS2/DDA data with centroided spectra, ideally one collision energy per precursor (the paper's benchmark sample holds ~4000 spectra). Load the sample mzML via 'Browse Sample.mzML' and an optional blank via 'Browse Blank.mzML' in the PFΔScreen GUI (installed in Python 3.9 with 'pip install -r requirements.txt'); a custom vendor feature list (external_feature_list.xlsx, preprocessed via 'Run ExternalFeatureTable') can replace pyOpenMS finding, but the corresponding mzML files must still be supplied or MS2 data would be missing. Execute 'Run FeatureFinding' with mass error 10 ppm, MS1 intensity threshold 2000 counts, and the OpenMS 'metabolites (5% RMS)' isotope model, keeping only peaks with FWHM above 1 s and below 1 min and at least two isotopic traces. Apply blank correction — remove features matching the extraction blank within <2 mDa at RT difference <0.1 min unless at least fivefold more abundant in the sample (soil M1 ESI−: 4209 features reduced to 3750). Align MS2 spectra at 5 mDa mass tolerance and 0.2 min RT tolerance so diagnostic fragments and fragment mass differences (±2 mDa, MS2 intensity threshold 2000 counts) can be matched downstream. The whole workflow runs below 1 min for ~4000 spectra, so iterate parameters and verify outputs before handing the feature/MS2 tables to 'Run PFASPrioritization'.

## Related tools

- **PFΔScreen (PFAScreen)** (GUI/console that loads the ddMS2 mzML sample and optional blank ('Browse Sample.mzML' / 'Browse Blank.mzML') and executes 'Run FeatureFinding' and 'Run PFASPrioritization') — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Python interface to OpenMS performing feature detection in the mzML raw data (mass error 10 ppm, MS1 intensity threshold 2000 counts, FWHM and isotope-trace filtering))
- **MSConvert** (Converts vendor raw MS data (Agilent .d) into vendor-independent mzML consumed by PFΔScreen)
- **OpenMS** (Supplies the 'metabolites (5% RMS)' isotope model used to exclude features with unusual peak shapes of isotopic traces)

## Examples

```
pip install -r requirements.txt && Run_PFAScreen.bat
```

## Evaluation signals

- Feature count in the expected range: on soil M1 (ESI−) the workflow detected 4209 features, reduced to 3750 after blank correction — empty results or orders-of-magnitude differences indicate mis-set parameters.
- Every surviving feature satisfies the FWHM window (>1 s and <1 min) and carries at least two isotopic traces under the metabolites (5% RMS) isotope model.
- Blank-corrected output contains no feature within <2 mDa and <0.1 min RT of a blank feature unless it is at least fivefold more abundant in the sample.
- MS2 linkage sanity check: in the paper's first iteration, 1026 of 2450 acquired MS2 spectra corresponded to detected features (417 unique, ~11% coverage); comparable coverage is expected for similar ddMS2 samples.
- Alignment respects the stated tolerances (5 mDa mass, 0.2 min RT) and whole-workflow runtime stays below 1 min for ~4000 spectra.

## Limitations

- Input must be ddMS2/DDA with centroided spectra, ideally one collision energy per precursor; profile-mode or non-DDA data fall outside the tool's validated scope.
- Data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing.
- The sub-1-minute runtime claim for ~4000 spectra is reported without hardware or computing environment specifications.
- No default or recommended input parameter values are documented; parameters must be adjusted per sample, instrument, and end-user needs.
- Automated filtering still requires manual verification of each feature for blank occurrence and reasonable peak shape (until <1% of the most abundant feature).

## Evidence

- [abstract] Feature detection operates on vendor-independent mzML from data-dependent acquisition via pyOpenMS: "Feature detection from vendor-independent MS raw data (mzML, data-dependent acquisition) is performed via pyOpenMS (or custom feature lists)"
- [results] Agilent raw .d files are converted to mzML with MSConvert before feature finding: "the raw MS data (.d files, Agilent) were converted into mzML with MSConvert"
- [results] Feature-finding parameters: 10 ppm mass error, 2000-count MS1 intensity threshold, OpenMS metabolites (5% RMS) isotope model: "The mass error for feature detection was set to 10 ppm, the MS1 intensity threshold was set to 2000 counts and the metabolites (5% RMS) isotope model from OpenMS was used"
- [results] Acceptance filter after feature detection: FWHM 1 s to 1 min and at least two isotopic traces: "Peaks reported after feature detection have to have a full width at half maximum (FWHM) above 1 s and below 1 min, and at least two isotopic traces."
- [results] MS2 spectra are aligned at 5 mDa mass tolerance and 0.2 min RT tolerance: "MS2 spectra were aligned with a mass tolerance of 5 mDa and an RT tolerance of 0.2 min"
- [readme] Sample and blank mzML inputs must come from ddMS2 acquisition with centroided spectra, one collision energy per precursor: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor."
- [readme] mzML files must accompany any feature table or MS2 data are lost during evaluation: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
- [readme] pyOpenMS is the engine for feature detection in raw MS data: "pyOpenMS (Python interface to the C++ OpenMS library) is used for feature detection in MS raw data."
