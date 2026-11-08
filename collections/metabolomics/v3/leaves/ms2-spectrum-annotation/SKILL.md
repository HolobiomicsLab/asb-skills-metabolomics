---
name: ms2-spectrum-annotation
description: Use when centroided ddMS2 mzML data must be inspected at the spectrum
  level — e.g., to verify whether PFAS diagnostic fragments or fragment mass differences
  were detected for a prioritized feature before assigning a suspect hit.
when_to_use_negative:
- The mzML lacks MS2 data (e.g., MS1-only acquisition or custom feature lists supplied
  without the corresponding mzML files) — MS2 extraction and annotation cannot work.
- Data are profile-mode or non-centroided, since PFΔScreen requires centroided ddMS2
  spectra with ideally one collision energy per precursor.
- The goal is de novo spectral library generation rather than overlaying PFΔScreen-detected
  diagnostic fragments and fragment mass differences.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: PFΔScreen
  role: Provides the RawDataVisualization tool whose MS2 extractor renders acquired
    MS2 spectra and overlays detected diagnostic fragments and fragment mass differences
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Used for feature detection on the mzML raw data so that MS2 spectra corresponding
    to detected features can be extracted and displayed
  repo: https://github.com/JonZwe/PFAScreen
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:60bd25035be4ce9eef5e771e4c8b41e58d865ca29c4eb76e2b465b93e1a7ba94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JonZwe/PFAScreen
  tools:
  - PFΔScreen
  - Python
  - pyOpenMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ms2-spectrum-annotation

## Summary

Reconstruction of PFΔScreen's MS2 spectrum extractor and annotation overlay: renders acquired ddMS2 spectra in the RawDataVisualization tool with detected diagnostic fragments and highlighted fragment mass differences so prioritized features can be structurally interpreted.

## When to use

Use when centroided ddMS2 mzML data must be inspected at the spectrum level — e.g., to verify whether PFAS diagnostic fragments or fragment mass differences were detected for a prioritized feature before assigning a suspect hit.

## When NOT to use

- The mzML lacks MS2 data (e.g., MS1-only acquisition or custom feature lists supplied without the corresponding mzML files) — MS2 extraction and annotation cannot work.
- Data are profile-mode or non-centroided, since PFΔScreen requires centroided ddMS2 spectra with ideally one collision energy per precursor.
- The goal is de novo spectral library generation rather than overlaying PFΔScreen-detected diagnostic fragments and fragment mass differences.

## Inputs

- Sample mzML raw data file (ddMS2, centroided spectra, e.g., converted from Agilent .d via MSConvert)
- Feature list for the feature of interest (from pyOpenMS feature finding or external_feature_list.xlsx)
- PFΔScreen results table (Excel) providing feature m/z, RT, and detected diagnostic fragments / fragment mass differences

## Outputs

- Rendered/interactive MS2 spectrum plots with overlaid and highlighted diagnostic fragments
- Highlighted fragment mass differences annotated on the MS2 spectra

## How to apply

First load the sample mzML (ddMS2, centroided, ideally one collision energy per precursor) via pyOpenMS feature finding, or supply an external feature list together with its corresponding mzML since MS2 data would otherwise be missing. After the PFASPrioritization tab has run, use the RawDataVisualization MS2 extractor to render the acquired MS2 spectra of the feature of interest. Detected diagnostic fragments and fragment mass differences are highlighted on the rendered spectra at a fragment-matching mass tolerance of ±2 mDa with an MS2 intensity threshold of 2000 counts. The annotation overlay lets the analyst judge whether MS2 evidence supports the PFAS class assignment (e.g., via diagnostic fragments such as CF2-related series) before pursuing EIC correlation or isotope-pattern checks. Validation consists of confirming that MS2 plots are produced end-to-end on a sample mzML and that highlighted fragments correspond to the detected diagnostic fragments/mass differences reported in the results table.

## Related tools

- **PFΔScreen** (Provides the RawDataVisualization tool whose MS2 extractor renders acquired MS2 spectra and overlays detected diagnostic fragments and fragment mass differences) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Used for feature detection on the mzML raw data so that MS2 spectra corresponding to detected features can be extracted and displayed) — https://github.com/JonZwe/PFAScreen

## Examples

```
double-click Run_PFAScreen.bat → FeatureFinding tab: Browse Sample.mzML, Run FeatureFinding → PFASPrioritization: Run PFASPrioritization → RawDataVisualization (MS2 extractor) to render the MS2 spectrum of a prioritized feature
```

## Evaluation signals

- The script runs end-to-end on a sample mzML and MS2 plots are produced for the feature of interest.
- Highlighted fragments on the rendered spectra fall within the ±2 mDa fragment-matching mass tolerance and above the 2000-count MS2 intensity threshold.
- Annotated diagnostic fragments and fragment mass differences match the entries in the PFΔScreen Excel results table.
- Annotation is only possible when the corresponding mzML file is supplied with the feature list (MS2 data otherwise missing) — absence of MS2 panels signals misconfiguration.
- For the M1 soil extract (first iteration), 417 unique MS2 spectra from 1026 feature-matched acquisitions formed the annotation pool, a sanity reference for expected MS2 coverage.

## Limitations

- Requires ddMS2 acquisition with centroided spectra and ideally one collision energy per precursor; data not meeting this cannot be annotated properly.
- Data evaluation only works when the corresponding mzML files are also given with external feature lists, otherwise MS2 data would be missing.
- Only about 11% MS2 coverage was achieved in the first iteration (417 of 2450 unique spectra), so many features will lack MS2 annotation.
- Diagnostic fragment/fragment-mass-difference detection depends on the article's fragment matching parameters (±2 mDa, 2000 counts); adjusting them changes the overlay.

## Evidence

- [readme] Afterwards, MS2 spectra displayed by the RawDataVisualization tool (MS2 extractor), have highlighted fragment mass differences and diagnostic fragments, if some were detected.: "Afterwards, MS2 spectra displayed by the RawDataVisualization tool (MS2 extractor), have highlighted fragment mass differences and diagnostic fragments, if some were detected."
- [results] Fragment/DF detection: mass tolerance ±2 mDa and MS2 intensity threshold 2000 counts: "at a mass tolerance set to ± 2 mDa and an MS2 intensity threshold of 2000 counts in the M1 soil extract (first iteration)"
- [readme] Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor.: "Sample and blank for raw data input in PFΔScreen should have been measured under data-dependent acquisition (ddMS2) with centroided spectra, ideally with one collision energy per precursor."
- [readme] Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing.: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
- [results] A total of 1026 out of 2450 acquired MS2 spectra corresponded to detected features, from which 417 unique spectra remained (~ 11% MS2 coverage in first iteration).: "A total of 1026 out of 2450 acquired MS2 spectra corresponded to detected features, from which 417 unique spectra remained (~ 11% MS2 coverage in first iteration)."
