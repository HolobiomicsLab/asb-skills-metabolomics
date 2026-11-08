---
name: homologous-series-eic-visualization
description: Use when prioritized PFAS features from non-target LC/GC-HRMS mzML data
  must be visually verified, homologous series EICs must be extrapolated from repeating
  units (e.g., CF2), or coeluting adduct/in-source-fragment ions must be flagged.
when_to_use_negative:
- Input is already a fully processed, deconvoluted peak table with no accompanying
  mzML file — MS2 spectra would be missing, so data evaluation will not work
- Raw data are profile-mode or not acquired under data-dependent acquisition with
  centroided spectra, contrary to PFΔScreen's requirements
- The goal is automated large-scale prioritization rather than visual inspection and
  verification of individual features — use the PFΔScreen PFASPrioritization tab instead
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PFΔScreen
  role: Provides the RawDataVisualization GUI whose EIC extractor, MS2 extractor and
    EIC correlator perform the visualization and coelution correlation
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Used for feature detection from mzML raw data so EIC/MS2 rendering can operate
    on detected features
- name: MSConvert
  role: Converts vendor raw files (e.g., Agilent .d) into the mzML format consumed
    by the visualization workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:d1775bc2e7fc687b6ada101895f1679218e88b69cb239d2c874970155043d2b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - MSConvert
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# homologous-series-eic-visualization

## Summary

Raw-data visualization workflow from PFΔScreen that extracts accurate-mass EICs from mzML MS1 data (including extrapolating multiple homologue EICs at once from a common repeating unit such as CF2), renders MS2 spectra with highlighted diagnostic fragments and fragment mass differences, and correlates coeluting EICs to flag in-source fragments and adducts.

## When to use

Use when prioritized PFAS features from non-target LC/GC-HRMS mzML data must be visually verified, homologous series EICs must be extrapolated from repeating units (e.g., CF2), or coeluting adduct/in-source-fragment ions must be flagged.

## When NOT to use

- Input is already a fully processed, deconvoluted peak table with no accompanying mzML file — MS2 spectra would be missing, so data evaluation will not work
- Raw data are profile-mode or not acquired under data-dependent acquisition with centroided spectra, contrary to PFΔScreen's requirements
- The goal is automated large-scale prioritization rather than visual inspection and verification of individual features — use the PFΔScreen PFASPrioritization tab instead

## Inputs

- Sample mzML raw data file (ddMS2, centroided spectra), e.g. converted from Agilent .d via MSConvert
- Optional blank-control mzML file
- External feature list spreadsheet (e.g., external_feature_list.xlsx / SampleFeatures.xlsx) with the corresponding mzML files
- PFΔScreen results table entries (m/z, RT) used to select EICs and spectra to visualize
- Common repeating unit definition for homologue extrapolation (e.g., CF2)

## Outputs

- EIC chromatograms (including multi-homologue series EICs from CF2-type repeating units)
- Interactive MS2 spectrum plots with highlighted diagnostic fragments and fragment mass differences
- Coelution correlation report of feature pairs with R² > 0.96 (candidate in-source fragments/adducts)
- Overlaid theoretical vs. experimental MS1 isotope pattern plots for suspect hits

## How to apply

Load the ddMS2 (centroided) sample mzML and, optionally, an external feature list (e.g., external_feature_list.xlsx) via pyOpenMS, keeping the corresponding mzML files present so MS2 data remain accessible. Extract accurate-mass EICs from MS1 at an extraction width of 5 mDa within an RT window of ±25 s, and extrapolate a series of homologue EICs at once from a common repeating unit such as CF2. Render acquired MS2 spectra overlaying diagnostic fragments and highlighted fragment mass differences at a fragment-matching tolerance of ±2 mDa with an MS2 intensity threshold of 2000 counts. Correlate coeluting EICs across the RT window and flag pairs with R² > 0.96 as candidate in-source fragments or adducts (e.g., 12 of 368 EICs correlated with R² > 0.96 at 5 mDa in the M1 soil extract). Optionally overlay theoretical isotope patterns of suspect hits onto the experimental MS1 isotope patterns.

## Related tools

- **PFΔScreen** (Provides the RawDataVisualization GUI whose EIC extractor, MS2 extractor and EIC correlator perform the visualization and coelution correlation) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Used for feature detection from mzML raw data so EIC/MS2 rendering can operate on detected features)
- **MSConvert** (Converts vendor raw files (e.g., Agilent .d) into the mzML format consumed by the visualization workflow)

## Examples

```
double-click Run_PFAScreen.bat → FeatureFinding tab: Browse Sample.mzML (+ Blank.mzML) → Run FeatureFinding → RawDataVisualization: enter m/z of interest, extract EICs (5 mDa, RT ±25 s) and run coelution correlation (R² > 0.96)
```

## Evaluation signals

- Script runs end-to-end on a sample mzML file and produces EIC and MS2 plots
- Coelution pairs are reported only when their correlation exceeds R² > 0.96 at the 5 mDa extraction width
- EIC extraction respects the 5 mDa mass width and ±25 s RT window; homologue series extrapolate from the specified repeating unit (e.g., CF2)
- MS2 spectra are rendered with fragment matches only at ±2 mDa tolerance and intensities above the 2000-count threshold
- Theoretical isotope patterns overlay plausibly on the experimental MS1 isotope patterns for suspect hits

## Limitations

- Custom feature lists only work when the corresponding mzML files are also provided, otherwise MS2 data would be missing
- Data should be ddMS2 with centroided spectra, ideally one collision energy per precursor
- MS2 fragment highlighting (diagnostic fragments / mass differences) only appears after the PFASPrioritization tab has been executed
- Reported correlation example covers only a subset (12 of 368 EICs above R² > 0.96), so the correlator is a flagging aid, not exhaustive adduct annotation

## Evidence

- [results] 12 out of 368 EICs correlated with an R2 > 0.96 at an extraction width of 5 mDa: "12 out of 368 EICs correlated with an R2 > 0.96 at an extraction width of 5 mDa"
- [results] Fragment/DF tolerance and threshold: "at a mass tolerance set to ± 2 mDa and an MS2 intensity threshold of 2000 counts in the M1 soil extract"
- [readme] Custom feature lists require mzML for MS2 access: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
- [readme] Coelution correlation and homologue EIC extrapolation described in README: "Data from the results table can be used to visualize EICs (and extrapolate HS with common repeating units such as CF2), MS1 and MS2 spectra. Additionally, a coelution correlation can be performed"
- [readme] Raw data acquisition requirements: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor."
