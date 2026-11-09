---
name: feature-labeling-convention-formatting
description: Use when feature tables from multiple separation groups (HILIC-MS, CE-MS)
  are being augmented into a single fused feature space and each row must retain its
  platform-of-origin identity via the 'm/z @Time, Group' label format.
when_to_use_negative:
- The analysis is single-platform (one separation technique only), so augmentation
  and group labeling add no information — tools like XCMS Online, MetaboAnalyst, MetaboAnalystR,
  or MZmine suffice for single-platform analysis
- Features need to be linked across methods by MS/MS spectra — AriumMS does not implement
  MS/MS spectral networking (listed as future development)
- The input is already an augmented, group-labeled feature table from a previous AriumMS
  run
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Hosts the augmentation routine that applies the 'm/z @Time, Group' labeling
    and merges matched cross-group features into augmented_feature_table.csv
  repo: https://github.com/AdrianHaun/AriumMS
- name: fangq/zmat
  role: ZMat toolbox dependency used by AriumMS's ROIpeaks/MSroiaug augmentation functions
    for data handling
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:9801999237ae1483d473ec97fa2a14efec89b29ab7488a4c6556100b104511ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AdrianHaun/AriumMS
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
yes_rate: 0.625
n_claims: 8
---

# feature-labeling-convention-formatting

## Summary

Applies AriumMS's 'm/z @Time, Group' naming convention during mid-level data fusion so that every feature in an augmented multi-platform (e.g. HILIC-MS + CE-MS) feature space remains attributable to its originating separation method before cross-group matching.

## When to use

Use when feature tables from multiple separation groups (HILIC-MS, CE-MS) are being augmented into a single fused feature space and each row must retain its platform-of-origin identity via the 'm/z @Time, Group' label format.

## When NOT to use

- The analysis is single-platform (one separation technique only), so augmentation and group labeling add no information — tools like XCMS Online, MetaboAnalyst, MetaboAnalystR, or MZmine suffice for single-platform analysis
- Features need to be linked across methods by MS/MS spectra — AriumMS does not implement MS/MS spectral networking (listed as future development)
- The input is already an augmented, group-labeled feature table from a previous AriumMS run

## Inputs

- Filtered per-group feature tables (one data cube per separation group, e.g. HILIC-MS and CE-MS) from the upstream false-positive filtering step
- m/z and time coordinates per feature from .mzXML/.mzML-derived AriumMS processing

## Outputs

- augmented_feature_table.csv with every row labeled 'm/z @Time, Group' and cross-group features merged on identical mass and occurrence count

## How to apply

Load the filtered per-group feature tables produced by the upstream false-positive filter, treating each separation group (e.g. HILIC-MS ESI positive/negative and nanoCEasy CE-MS) as a separate data cube. Run the AriumMS augmentation routine, which relies on the MSroiaug function of Tauler/Gorrochategui/Jaumot (ROI-based augmentation) to link the per-group cubes along the m/z dimension and combine m/z and time into a single fused dimension. Label every feature in the augmented space using the 'm/z @Time, Group' convention. Then match features across groups sharing an identical mass and identical occurrence count, collapsing them into single augmented entries, and write the result to augmented_feature_table.csv. The rationale is that without group-qualified labels, features from different separation methods with identical nominal characteristics could not be distinguished or traced back to their platform.

## Related tools

- **AriumMS** (Hosts the augmentation routine that applies the 'm/z @Time, Group' labeling and merges matched cross-group features into augmented_feature_table.csv) — https://github.com/AdrianHaun/AriumMS
- **fangq/zmat** (ZMat toolbox dependency used by AriumMS's ROIpeaks/MSroiaug augmentation functions for data handling) — http://github.com/fangq/zmat

## Evaluation signals

- Every row of augmented_feature_table.csv carries the 'm/z @Time, Group' label format
- Features from different groups with identical mass and identical occurrence count have been merged into single augmented entries
- The augmented table spans the combined m/z and time dimensions of all input separation groups
- No feature lost its attribution to its originating separation method after fusion (labels disambiguate same-mass features from different platforms)

## Limitations

- Cross-group matching collapses only features with identical mass and occurrence count; MS/MS spectral networking to autonomously connect features across methods is not implemented
- Augmentation is currently limited to chromatographic/electrophoretic MS data; augmenting spectroscopic data is only mentioned as possible future work
- The AriumMS app requires MATLAB R2024a or newer plus several toolboxes (Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, Database)

## Evidence

- [full_text] m/z @Time, Group label convention during augmentation: [evidence span withheld — anchor status: not_found]
- [discussion] augmentation step in the AriumMS pipeline: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [readme] MSroiaug function underlying augmentation: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [discussion] mid-level data fusion purpose: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset"
