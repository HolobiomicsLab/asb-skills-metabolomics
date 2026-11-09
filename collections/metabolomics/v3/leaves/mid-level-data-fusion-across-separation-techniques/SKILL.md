---
name: mid-level-data-fusion-across-separation-techniques
description: Use when untargeted metabolomics feature tables from different separation
  platforms (e.g., HILIC-Orbitrap MS ESI+/- and CE-Orbitrap MS) must be combined into
  one augmented dataset for joint statistical evaluation.
when_to_use_negative:
- Data are single-platform only — existing tools like XCMS Online, MetaboAnalyst,
  MetaboAnalystR or MZmine handle single-platform analysis, and augmentation of different
  separation techniques adds no value.
- Features have not yet passed the upstream false-positive filter, since the augmentation
  step expects the filtered per-group data cubes as input.
- Features from two or more methods need to be connected precisely and autonomously
  via MS/MS spectra — AriumMS does not implement MS/MS spectral networking (listed
  as future development).
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Provides the augmentation routine (MSroiaug-based) that links per-group data
    cubes along the m/z dimension, applies the 'm/z @Time, Group' feature naming,
    and merges cross-group features with identical mass and occurrence count.
  repo: https://github.com/AdrianHaun/AriumMS
- name: zmat
  role: ZMat toolbox dependency used by AriumMS for compressed data handling within
    the MATLAB processing pipeline.
  repo: http://github.com/fangq/zmat
- name: msconvert (ProteoWizard)
  role: Used upstream to convert vendor MS data to .mzXML or .mzML file format before
    ROI search, preprocessing, and augmentation.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:388f785ff02dba07edcce98b494f42383458fd6dec6b2aa0c0dd025cf32e3c5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AdrianHaun/AriumMS
  tools:
  - AriumMS
  - msconvert (ProteoWizard)
  - zmat
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mid-level-data-fusion-across-separation-techniques

## Summary

AriumMS's mid-level data fusion links per-separation-method feature data cubes (e.g., HILIC-MS and CE-MS) along the m/z dimension into a single augmented feature space, collapsing cross-group features with identical mass and occurrence count so multi-platform data can be evaluated with one chemometric result.

## When to use

Use when untargeted metabolomics feature tables from different separation platforms (e.g., HILIC-Orbitrap MS ESI+/- and CE-Orbitrap MS) must be combined into one augmented dataset for joint statistical evaluation.

## When NOT to use

- Data are single-platform only — existing tools like XCMS Online, MetaboAnalyst, MetaboAnalystR or MZmine handle single-platform analysis, and augmentation of different separation techniques adds no value.
- Features have not yet passed the upstream false-positive filter, since the augmentation step expects the filtered per-group data cubes as input.
- Features from two or more methods need to be connected precisely and autonomously via MS/MS spectra — AriumMS does not implement MS/MS spectral networking (listed as future development).

## Inputs

- Filtered per-group feature tables from the upstream false-positive filtering step, one group per separation method (e.g., HILIC-MS ESI positive, HILIC-MS ESI negative, three CE-Orbitrap MS methods)
- Raw MS data previously converted to .mzXML or .mzML via ProteoWizard msconvert

## Outputs

- augmented_feature_table.csv — augmented, group-labeled feature table spanning the combined m/z and time dimensions of all input groups

## How to apply

Load the filtered per-group feature tables produced by the upstream false-positive filter, treating each separation group (e.g., HILIC-MS and CE-MS) as a separate data cube. Run the AriumMS augmentation routine (built on the ROIpeaks/MSroiaug functions of Tauler, Gorrochategui and Jaumot) to link the per-group data cubes along the m/z dimension, combining the m/z and time dimensions into a single fused dimension. Label every feature in the augmented space with the 'm/z @Time, Group' convention so features stay attributable to their originating separation method, then match features across groups sharing identical mass and identical occurrence count and collapse them into single augmented entries. Write the fused table to augmented_feature_table.csv; this mid-level fusion also removes artifacts and noise from the combined LC-MS + CE-MS dataset.

## Related tools

- **AriumMS** (Provides the augmentation routine (MSroiaug-based) that links per-group data cubes along the m/z dimension, applies the 'm/z @Time, Group' feature naming, and merges cross-group features with identical mass and occurrence count.) — https://github.com/AdrianHaun/AriumMS
- **zmat** (ZMat toolbox dependency used by AriumMS for compressed data handling within the MATLAB processing pipeline.) — http://github.com/fangq/zmat
- **msconvert (ProteoWizard)** (Used upstream to convert vendor MS data to .mzXML or .mzML file format before ROI search, preprocessing, and augmentation.)

## Examples

```
MATLAB R2024a: launch AriumMS app, load filtered per-group feature tables (HILIC-MS and CE-MS groups), run MSroiaug-based augmentation to write augmented_feature_table.csv
```

## Evaluation signals

- Every row of augmented_feature_table.csv carries the 'm/z @Time, Group' label format.
- Matched cross-group features with identical mass and identical occurrence count have been merged into single augmented entries.
- The augmented table spans the combined m/z and time dimensions of all input separation groups (HILIC-MS + CE-MS).
- Downstream chemometric evaluation operates on a single fused dataset rather than separate per-platform tables.
- The fused dataset demonstrably removes artifacts and noise relative to the unfused LC-MS + CE-MS tables, as reported for yeast extract mid-level fusion.

## Limitations

- MS/MS spectral networking is not implemented; autonomous connection of features detected by two or more methods via MS/MS is planned future development.
- Augmentation of spectroscopic data with chromatographic/electrophoretic data is not supported (mentioned as possible future work).
- Requires MATLAB R2024a or newer with Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes.
- AriumMS processes multi-platform data by augmentation but focuses on polar metabolites analyzed with HILIC-MS/CE-MS techniques.

## Evidence

- [abstract] AriumMS offers augmented data analysis of several separation techniques utilizing a region-of-interest algorithm: "AriumMS offers augmented data analysis of several separation techniques utilizing a region-of-interest algorithm"
- [discussion] AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [readme] Set parameters are then used to perform ROI search, data preprocessing and data augmentation: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation. It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [readme] For MS Data conversion to .mzXML or .mzML file format use msconvert: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
- [discussion] AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
