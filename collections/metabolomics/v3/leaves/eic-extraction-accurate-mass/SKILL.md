---
name: eic-extraction-accurate-mass
description: Use when you need to visualize and validate prioritized PFAS features
  in mzML raw data — e.g., extract EICs for suspects or homologue series, correlate
  coeluting EICs to flag in-source fragments/adducts, or overlay MS2 diagnostic fragments
  and isotope patterns.
when_to_use_negative:
- The mzML data are not centroided or were not acquired with data-dependent acquisition
  (ddMS2) with ideally one collision energy per precursor — MS2 spectra will be missing
  or unusable
- Only a feature table is available without the corresponding mzML files — data evaluation
  fails because MS2 data would be missing
- The goal is bulk automated prioritization across many features rather than interactive
  visualization/validation of specific hits — use the PFASPrioritization tab instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Provides the RawDataVisualization GUI (EIC extractor, MS2 extractor, EIC correlator)
    used to extract EICs, render MS2 spectra, and perform coelution correlation
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Python interface to OpenMS used for feature detection in MS raw data and for
    accessing MS1/MS2 spectra from mzML files
  repo: https://github.com/JonZwe/PFAScreen
- name: MSConvert
  role: Converts vendor raw MS data (e.g., Agilent .d) into the mzML format consumed
    by the EIC/MS2 extraction workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:58f15c922b609f27067e0f7693d95abef334fb6b195805d184a4c6cb3aed57a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# eic-extraction-accurate-mass

## Summary

Accurate-mass extracted ion chromatogram (EIC) extraction, rendering, and coelution correlation from ddMS2 mzML raw data as implemented in PFΔScreen's RawDataVisualization tool (EIC extractor, MS2 extractor, EIC correlator). Used to visualize features of interest, extrapolate homologue series (e.g., CF2), flag coeluting in-source fragments/adducts, and overlay theoretical isotope patterns on experimental MS1.

## When to use

Use when you need to visualize and validate prioritized PFAS features in mzML raw data — e.g., extract EICs for suspects or homologue series, correlate coeluting EICs to flag in-source fragments/adducts, or overlay MS2 diagnostic fragments and isotope patterns.

## When NOT to use

- The mzML data are not centroided or were not acquired with data-dependent acquisition (ddMS2) with ideally one collision energy per precursor — MS2 spectra will be missing or unusable
- Only a feature table is available without the corresponding mzML files — data evaluation fails because MS2 data would be missing
- The goal is bulk automated prioritization across many features rather than interactive visualization/validation of specific hits — use the PFASPrioritization tab instead

## Inputs

- Sample mzML raw data file (ddMS2, centroided spectra; e.g., converted from Agilent .d with MSConvert)
- Optional blank mzML file for control comparison
- Feature list (e.g., external_feature_list.xlsx / SampleFeatures.xlsx) from PFΔScreen feature finding or vendor software
- PFΔScreen results table entries (m/z, RT) used as seeds for EIC visualization

## Outputs

- Interactive EIC plots of the m/z of interest and extrapolated homologue series (e.g., CF2 repeating unit)
- Rendered MS2 spectra with highlighted diagnostic fragments and fragment mass differences (±2 mDa, 2000 counts threshold)
- Coelution correlation report listing EIC pairs correlated above R² > 0.96 as candidate in-source fragments or adducts
- Overlays of theoretical isotope patterns on experimental MS1 isotope patterns for suspect hits

## How to apply

Load a sample mzML (ddMS2, centroided spectra, ideally one collision energy per precursor) plus a feature list such as external_feature_list.xlsx via pyOpenMS in PFΔScreen's FeatureFinding tab; data evaluation only works when the corresponding mzML files accompany the feature tables, otherwise MS2 data would be missing. Implement accurate-mass EIC extraction from MS1 at an extraction width of 5 mDa within an RT window of ±25 s, supporting extraction of multiple homologue EICs at once by extrapolating a series from a common repeating unit such as CF2. Implement an MS2 spectrum extractor that renders acquired MS2 spectra and overlays detected diagnostic fragments and highlighted fragment mass differences at a fragment-matching mass tolerance of ±2 mDa and an MS2 intensity threshold of 2000 counts. Correlate the EICs of all detected features against a user-specified m/z of interest over the RT range, flagging pairs with R² > 0.96 as candidate in-source fragments or adducts (e.g., 12 out of 368 EICs correlated at R² > 0.96 at 5 mDa). Display theoretical isotope patterns of suspect hits over the experimental MS1 isotope patterns to complete the visual validation.

## Related tools

- **PFΔScreen** (Provides the RawDataVisualization GUI (EIC extractor, MS2 extractor, EIC correlator) used to extract EICs, render MS2 spectra, and perform coelution correlation) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Python interface to OpenMS used for feature detection in MS raw data and for accessing MS1/MS2 spectra from mzML files) — https://github.com/JonZwe/PFAScreen
- **MSConvert** (Converts vendor raw MS data (e.g., Agilent .d) into the mzML format consumed by the EIC/MS2 extraction workflow)

## Evaluation signals

- EICs are extracted at an extraction width of 5 mDa within an RT window of ±25 s from the MS1 data of the mzML file
- Coelution pairs are reported only when their correlation exceeds R² > 0.96 at 5 mDa extraction width (reference behavior: 12 of 368 EICs at R² > 0.96)
- MS2 spectra are annotated only for fragments within ±2 mDa and above an intensity threshold of 2000 counts
- Homologue series extrapolation correctly steps a common repeating unit (e.g., CF2) across multiple simultaneously extracted EICs
- The script/plot pipeline runs end-to-end on a sample mzML and produces EIC plots, MS2 plots, and isotope-pattern overlays without missing MS2 data

## Limitations

- Data evaluation only works when the corresponding mzML files are provided together with external feature lists; otherwise MS2 data would be missing
- Raw data input should be ddMS2 with centroided spectra, ideally one collision energy per precursor — other acquisition modes are not supported by the MS2 extractor
- Correlation-based coelution flags (R² > 0.96) are candidates for in-source fragments or adducts and still require manual verification of peak shape and blank occurrence

## Evidence

- [results] 12 out of 368 EICs correlated with an R2 > 0.96 at an extraction width of 5 mDa: "12 out of 368 EICs correlated with an R2 > 0.96 at an extraction width of 5 mDa"
- [readme] README: mzML requirement and missing MS2: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
- [readme] README: EIC visualization and homologue extrapolation: "Data from the results table can be used to visualize EICs (and extrapolate HS with common repeating units such as CF2), MS1 and MS2 spectra."
- [readme] README: ddMS2 centroided input requirement: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor."
- [abstract] Abstract: pyOpenMS feature detection and mzML input: "Feature detection from vendor-independent MS raw data (mzML, data-dependent acquisition) is performed via pyOpenMS (or custom feature lists)"
