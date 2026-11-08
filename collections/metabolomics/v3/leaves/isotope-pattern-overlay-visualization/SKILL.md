---
name: isotope-pattern-overlay-visualization
description: Use when inspecting PFΔScreen-prioritized suspect-screening hits whose
  accurate mass matched within 4 mDa, to visually confirm that the experimental MS1
  isotopologue envelope agrees with the theoretical pattern before accepting a candidate
  PFAS annotation.
when_to_use_negative:
- The input is only a feature table without the corresponding mzML file — the README
  states data evaluation only works when the corresponding mzML files are also given,
  otherwise MS2/MS1 raw data would be missing
- The feature has not been through feature detection or external feature-list preprocessing
  (Run FeatureFinding / Run ExternalFeatureTable), so no experimental MS1 isotope
  pattern exists to overlay
- The goal is automated filtering rather than visual verification — isotope-pattern
  plausibility during feature detection is handled by the OpenMS metabolites (5% RMS)
  isotope model, not by manual overlay
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: PFΔScreen
  role: Provides the RawDataVisualization tool that displays theoretical isotope patterns
    of suspect hits over the experimental MS1 isotope patterns
  repo: https://github.com/JonZwe/PFAScreen
- name: pyOpenMS
  role: Used for feature detection from mzML raw data, supplying the detected features
    and MS1 isotope traces that are visualized
  repo: https://github.com/JonZwe/PFAScreen
- name: OpenMS
  role: Source of the metabolites (5% RMS) isotope model used for isotope-trace plausibility
    in the workflow
- name: MSConvert
  role: Converts vendor raw data (e.g., Agilent .d files) into mzML with centroided
    ddMS2 spectra required as input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-05070-2
    title: PFΔScreen — an open-source tool for automated PFAS feature prioritization
      in non-target HRMS data
  source_content_hash: sha256:d7d8c93aedbe998e3ecdf47b5fad4250a2261dc711a90f832648001beefcc063
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# isotope-pattern-overlay-visualization

## Summary

Visualization skill from PFΔScreen's RawDataVisualization tool that overlays the theoretical isotope pattern of suspect hits onto the experimental MS1 isotope pattern, helping to judge whether a prioritized feature's isotope envelope is consistent with the proposed PFAS assignment.

## When to use

Use when inspecting PFΔScreen-prioritized suspect-screening hits whose accurate mass matched within 4 mDa, to visually confirm that the experimental MS1 isotopologue envelope agrees with the theoretical pattern before accepting a candidate PFAS annotation.

## When NOT to use

- The input is only a feature table without the corresponding mzML file — the README states data evaluation only works when the corresponding mzML files are also given, otherwise MS2/MS1 raw data would be missing
- The feature has not been through feature detection or external feature-list preprocessing (Run FeatureFinding / Run ExternalFeatureTable), so no experimental MS1 isotope pattern exists to overlay
- The goal is automated filtering rather than visual verification — isotope-pattern plausibility during feature detection is handled by the OpenMS metabolites (5% RMS) isotope model, not by manual overlay

## Inputs

- PFΔScreen results table (Excel) from the PFASPrioritization tab, containing suspect hits and their m/z and RT
- Sample mzML raw data file (ddMS2, centroided) corresponding to the feature list
- Suspect list entry with theoretical formula/isotope pattern for the hit (e.g., from PFASMASTER / PFASOECDNA suspects)

## Outputs

- Interactive plot overlaying the theoretical isotope pattern of the suspect hit on the experimental MS1 isotope pattern
- Visual confirmation record for the suspect-hit annotation (part of the interactive HTML plots folder)

## How to apply

After the PFASPrioritization tab has been executed and suspect hits have been filtered by accurate mass (tolerance of 4 mDa), open the results table saved in the sample-named output folder and use the RawDataVisualization tool to plot the MS1 isotope pattern of the feature of interest. Overlay the theoretical isotope pattern of the corresponding suspect hit onto the experimental MS1 pattern; the article notes that the metabolites (5% RMS) isotope model from OpenMS is the reference model used elsewhere in the workflow for isotope-trace plausibility. Inspect whether the isotope envelopes (spacing and relative intensities, e.g., CF2-homologue envelopes) coincide; features with unusual isotope-trace peak shapes were excluded during feature finding, so the overlay is the last visual confirmation step. Treat agreement of the overlay as supporting (not standalone) evidence, to be combined with MS2 diagnostic fragments, KMD homologous series, and MD/C-m/C prioritization before reporting the annotation.

## Related tools

- **PFΔScreen** (Provides the RawDataVisualization tool that displays theoretical isotope patterns of suspect hits over the experimental MS1 isotope patterns) — https://github.com/JonZwe/PFAScreen
- **pyOpenMS** (Used for feature detection from mzML raw data, supplying the detected features and MS1 isotope traces that are visualized) — https://github.com/JonZwe/PFAScreen
- **OpenMS** (Source of the metabolites (5% RMS) isotope model used for isotope-trace plausibility in the workflow)
- **MSConvert** (Converts vendor raw data (e.g., Agilent .d files) into mzML with centroided ddMS2 spectra required as input)

## Examples

```
double click Run_PFAScreen.bat, run PFASPrioritization, then in RawDataVisualization load the sample results table and select a suspect hit to overlay its theoretical isotope pattern on the experimental MS1 pattern
```

## Evaluation signals

- The overlay plot is produced only after the PFASPrioritization tab has run and a results table (Excel) exists in the sample-named output folder
- The suspect hit being overlaid passed accurate-mass suspect screening at a tolerance of 4 mDa (e.g., hits reduced from 217 to 176)
- The experimental MS1 pattern shows the expected isotopologue spacing and at least two isotopic traces (features required FWHM above 1 s and below 1 min with ≥2 isotopic traces)
- The theoretical envelope aligns with the experimental envelope in relative intensity, consistent with fluorine-dominated composition (m/C typically > 30)
- End-to-end run on a sample mzML produces the interactive HTML plot without errors, alongside the other RawDataVisualization outputs (EIC, MS1, MS2 spectra)

## Limitations

- Overlay agreement is a supporting visual check, not a stand-alone identification criterion; suspect hits are only accurate-mass (4 mDa) matches and must be combined with MS2 fragment and KMD evidence
- Data evaluation requires the corresponding mzML files to be provided together with feature lists, otherwise raw MS1/MS2 data are missing and the visualization cannot be performed
- The OpenMS isotope model (metabolites, 5% RMS) used for isotope-trace plausibility may exclude unusual but real isotope peak shapes during feature detection, biasing what can later be overlaid

## Evidence

- [readme] theoretical isotope patterns of suspect hits can be displayed over the experimental isotope patterns (MS1): "Also, the theoretical isotope patterns of suspect hits can be displayed over the experimental isotope patterns (MS1)."
- [results] suspect screening accurate mass tolerance of 4 mDa: "the hits by accurate mass (tolerance of 4 mDa) were reduced from 217 to 176"
- [results] OpenMS isotope model for isotope traces: "the metabolites (5% RMS) isotope model from OpenMS was used to exclude features with unusual peak shapes of isotopic traces"
- [readme] mzML requirement for MS2/raw data access: "Note that data evaluation only works when the corresponding mzML files are also given; otherwise MS2 data would be missing."
