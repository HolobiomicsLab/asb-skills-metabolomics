---
name: feature-blank-correction-filtering
description: Use when a sample mzML has a paired extraction blank mzML and detected
  features must be filtered for background contamination before PFAS prioritization
  (MD/C-m/C, KMD, suspect screening).
when_to_use_negative:
- No extraction blank mzML is available or the blank was not measured under the same
  ddMS2 conditions — blank correction logic cannot be applied
- Input is already a vendor feature table intended for the external-feature-list path
  ('Browse SampleFeatures.xlsx' / 'Run ExternalFeatureTable') where blank features
  must be supplied as BlankFeatures.xlsx rather than filtered from raw mzML
- Goal is raw-data visualization only via RawDataVisualization, which works without
  PFAS-specific blank correction
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Hosts the GUI and implements the blank-correction filter (2 mDa / 0.1 min
    RT / 5-fold abundance rule) within its FeatureFinding-to-PFASPrioritization workflow
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Performs the feature detection on sample and blank mzML whose outputs are
    compared for blank correction
  repo: https://github.com/JonZwe/PFAScreen
- name: MSConvert
  role: Converts vendor raw data (e.g., Agilent .d) into the mzML files consumed by
    PFΔScreen for sample and blank
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:0e6ec50040f43302c4ad37591d5d634d1ca0a67194a6249534444899c9a06c5b
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# feature-blank-correction-filtering

## Summary

Blank-correction step of the PFΔScreen workflow that removes background/container features detected in both a ddMS2 mzML sample and its extraction blank, reducing non-target HRMS feature lists (e.g., 4209 to 3750 features in soil M1, ESI−) before PFAS-specific prioritization.

## When to use

Use when a sample mzML has a paired extraction blank mzML and detected features must be filtered for background contamination before PFAS prioritization (MD/C-m/C, KMD, suspect screening).

## When NOT to use

- No extraction blank mzML is available or the blank was not measured under the same ddMS2 conditions — blank correction logic cannot be applied
- Input is already a vendor feature table intended for the external-feature-list path ('Browse SampleFeatures.xlsx' / 'Run ExternalFeatureTable') where blank features must be supplied as BlankFeatures.xlsx rather than filtered from raw mzML
- Goal is raw-data visualization only via RawDataVisualization, which works without PFAS-specific blank correction

## Inputs

- Sample ddMS2 mzML file (centroided spectra, vendor-independently converted e.g. via MSConvert)
- Extraction blank mzML file (same DDA acquisition mode, optional but required for blank correction)
- Detected feature lists from pyOpenMS feature finding for sample and blank

## Outputs

- Blank-corrected feature list passed downstream to PFAS prioritization (MD/C-m/C, KMD, MS2 fragment matching, suspect screening)
- PFΔScreen results table (Excel) and interactive HTML plots (MD/C-m/C plot, m/z vs. RT plot, KMD vs. m/z, m/C histogram) after full workflow execution

## How to apply

Run PFΔScreen's FeatureFinding (pyOpenMS feature detection: 10 ppm mass error, MS1 intensity threshold 2000 counts, OpenMS metabolites 5% RMS isotope model, FWHM 1 s–1 min with at least two isotopic traces) separately on the sample and the extraction blank mzML files, both acquired in data-dependent acquisition with centroided spectra. Then apply the blank-correction logic: any feature present in both sample and blank that deviates by < 2 mDa in mass at an RT difference of < 0.1 min is removed unless it is at least fivefold more abundant in the sample than in the blank. In the GUI this is controlled via the FeatureFinding tab (Browse Sample.mzML / Browse Blank.mzML) with parameters for feature finding, MS2 alignment (5 mDa mass tolerance, 0.2 min RT tolerance), and blank correction, executed before 'Run PFASPrioritization'. The rationale is that soil/sediment extracts and their blanks share many background signals, so blank subtraction drastically cuts the feature list before the PFAS-specific MD/C-m/C reduction (m/C cutoff 30, MD/C < +0.003) and KMD analysis.

## Related tools

- **PFΔScreen** (Hosts the GUI and implements the blank-correction filter (2 mDa / 0.1 min RT / 5-fold abundance rule) within its FeatureFinding-to-PFASPrioritization workflow) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Performs the feature detection on sample and blank mzML whose outputs are compared for blank correction) — https://github.com/JonZwe/PFAScreen
- **MSConvert** (Converts vendor raw data (e.g., Agilent .d) into the mzML files consumed by PFΔScreen for sample and blank)

## Examples

```
pip install -r requirements.txt; python Run_PFAScreen.bat  # GUI: Browse Sample.mzML + Browse Blank.mzML -> Run FeatureFinding (blank correction) -> Run PFASPrioritization
```

## Evaluation signals

- Feature count drops after blank correction (paper example: 4209 features reduced to 3750 in soil M1, ESI−)
- Retained features not matching blank within 2 mDa / 0.1 min RT are kept; matched features are removed unless ≥5-fold more abundant in sample
- Corrected feature list feeds PFAS prioritization and yields expected downstream results (e.g., KMD homologous series with ≥3 homologues at ±2 mDa, suspect hits at 4 mDa tolerance)
- Whole workflow including blank correction completes in <1 min for ~4000 spectra per sample

## Limitations

- Blank correction removes features not at least fivefold more abundant in the sample; weak genuine PFAS signals near blank levels may be lost and require the paper's manual verification step (blank occurrence and peak shape until <1% of most abundant feature)
- The rule is tuned to extraction blanks measured alongside each soil sample; blanks measured under different conditions or without DDA/centroided spectra may misalign
- Runtime/hardware environment and default parameter values are not fully specified in the paper; parameters must be adjusted per sample and instrument

## Evidence

- [results] Features detected in both sample and extraction blank that deviated by < 2 mDa at a RT difference of < 0.1 min and were not at least fivefold more abundant in the sample were removed.: "Features detected in both sample and extraction blank that deviated by < 2 mDa at a RT difference of < 0.1 min and were not at least fivefold more abundant in the sample were removed."
- [results] Exemplified on soil M1, 4209 features were detected, which were reduced to 3750 features after blank correction in the ESI− mode.: "Exemplified on soil M1, 4209 features were detected, which were reduced to 3750 features after blank correction in the ESI− mode."
- [readme] To load a MS raw datafile, click the “Browse Sample.mzML” button and choose the mzML file of a sample and an optional mzML file of a blank control (Browse Blank.mzML).: "To load a MS raw datafile, click the “Browse Sample.mzML” button and choose the mzML file of a sample and an optional mzML file of a blank control (Browse Blank.mzML)."
- [results] For each soil, PFΔScreen was executed individually together with the extraction blank to remove background signals: "For each soil, PFΔScreen was executed individually together with the extraction blank to remove background signals"
