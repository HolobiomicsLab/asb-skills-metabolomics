---
name: cross-group-feature-matching
description: Use when two or more feature tables from different separation platforms
  (e.g. HILIC-MS ESI+/ESI- and CE-MS via nanoCEasy) must be fused into one augmented
  dataset for joint chemometric evaluation.
when_to_use_negative:
- Your analysis targets a single separation platform — existing tools like XCMS Online,
  MetaboAnalyst, MetaboAnalystR or MZmine handle single-platform processing without
  augmentation
- You need to connect features across methods by MS/MS evidence — MS/MS spectral networking
  is not implemented in AriumMS and is listed only as future development
- You want to augment spectroscopic data with chromatographic/electrophoretic data
  — this is mentioned as possibly interesting future work, not implemented
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Provides the augmentation routine that links per-group data cubes along m/z,
    applies 'm/z @Time, Group' labeling, and merges cross-group features with identical
    mass and occurrence count
  repo: github.com/AdrianHaun/AriumMS
- name: fangq/zmat
  role: ZMat toolbox dependency used by AriumMS for compressed data handling in the
    MATLAB pipeline
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:4011f8b1d6ef0ff4d033aa1c0514a5ec531ebb01457d888f4d63628741d40836
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - fangq/zmat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-group-feature-matching

## Summary

Mid-level data fusion step of AriumMS that links per-separation-group data cubes (e.g. HILIC-MS and CE-MS) along the m/z dimension, labels features with the 'm/z @Time, Group' convention, and merges cross-group features sharing identical mass and occurrence count into a single augmented feature table.

## When to use

Use when two or more feature tables from different separation platforms (e.g. HILIC-MS ESI+/ESI- and CE-MS via nanoCEasy) must be fused into one augmented dataset for joint chemometric evaluation.

## When NOT to use

- Your analysis targets a single separation platform — existing tools like XCMS Online, MetaboAnalyst, MetaboAnalystR or MZmine handle single-platform processing without augmentation
- You need to connect features across methods by MS/MS evidence — MS/MS spectral networking is not implemented in AriumMS and is listed only as future development
- You want to augment spectroscopic data with chromatographic/electrophoretic data — this is mentioned as possibly interesting future work, not implemented

## Inputs

- Filtered per-group feature tables (one data cube per separation group, e.g. HILIC-MS and CE-MS) from the upstream false-positive filtering step
- User-defined group definitions (one group per separation method)
- MS data previously converted to .mzXML or .mzML via msconvert (ProteoWizard) in the upstream pipeline

## Outputs

- augmented_feature_table.csv: augmented, group-labeled feature table spanning the combined m/z and time dimensions of all input groups, with cross-group features of identical mass and occurrence count merged

## How to apply

Load the filtered per-group feature tables produced by the upstream false-positive filter, treating each separation group (e.g. HILIC-MS and CE-MS) as a separate data cube. Run the AriumMS augmentation routine (the MSroiaug function from Tauler/Gorrochategui/Jaumot) to link the per-group cubes along the m/z dimension, combining the m/z and time dimensions into a single augmented feature space. Label every feature using the 'm/z @Time, Group' convention so each feature remains attributable to its originating separation method. Match features across groups that share an identical mass and identical occurrence count, collapsing them into single augmented entries — this is the mid-level fusion of the LC-MS + CE-MS tables. Write the result to augmented_feature_table.csv and check that matched cross-group features were merged and all rows carry the group label. The rationale is that fused data yield a single chemometric result with enhanced statistical prediction and metabolic coverage, and simultaneously remove artifacts and noise from the entire analytical dataset.

## Related tools

- **AriumMS** (Provides the augmentation routine that links per-group data cubes along m/z, applies 'm/z @Time, Group' labeling, and merges cross-group features with identical mass and occurrence count) — github.com/AdrianHaun/AriumMS
- **fangq/zmat** (ZMat toolbox dependency used by AriumMS for compressed data handling in the MATLAB pipeline) — http://github.com/fangq/zmat

## Evaluation signals

- Every row of augmented_feature_table.csv carries the 'm/z @Time, Group' label format
- Cross-group features with identical mass and identical occurrence count are merged into single augmented entries
- The augmented table spans the combined m/z and time dimensions of all input groups (HILIC-MS + CE-MS)
- Fused dataset removes artifacts and noise, as validated in the paper by mid-level data fusion on the LC-MS + CE-MS yeast extract dataset
- Downstream chemometric evaluation on the fused table reproduces the paper's result of differentiating yeast metabolomes treated with a growth inhibitor

## Limitations

- Features from different methods are only linked by identical mass and occurrence count; MS/MS spectral networking for precise autonomous feature connection is not yet implemented
- Augmentation of spectroscopic data with chromatographic/electrophoretic data is not supported (future work)
- Requires MATLAB R2024a or newer plus Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing and Database toolboxes (app version)

## Evidence

- [abstract] Augmented data analysis of several separation techniques using a region-of-interest algorithm: "AriumMS offers augmented data analysis of several separation techniques utilizing a region-of-interest algorithm"
- [discussion] Mid-level data fusion removes artifacts and noise from the combined LC-MS + CE-MS dataset: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [intro] Rationale: a single fused chemometric result improves prediction and coverage: "a combined multi-platform non-targeted data evaluation based on a data fusion approach offers a single chemometric result for enhanced statistical prediction and metabolic coverage"
- [readme] Augmentation relies on ROIpeaks and MSroiaug functions within the MATLAB AriumMS app: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
