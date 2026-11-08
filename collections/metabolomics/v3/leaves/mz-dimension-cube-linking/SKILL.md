---
name: mz-dimension-cube-linking
description: Use when filtered per-group feature tables from different separation
  platforms (e.g., HILIC-MS ESI+/ESI- and CE-MS via the nanoCEasy interface) must
  be fused into a single augmented 'm/z @Time, Group' feature space for joint chemometric
  evaluation.
when_to_use_negative:
- 'Single-platform analysis: existing tools like XCMS Online, MetaboAnalyst, MetaboAnalystR,
  or MZmine suffice when only one separation technique is involved and no augmentation
  across methods is needed'
- When precise autonomous linkage of features detected by two or more methods via
  MS/MS is required — MS/MS spectral networking is not implemented in AriumMS and
  is listed as future development
- 'Input is raw vendor MS data: files must first be converted to .mzXML or .mzML with
  msconvert (ProteoWizard) before the AriumMS pipeline, and augmentation operates
  downstream of ROI search, preprocessing, feature detection, and false-positive filtering'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Toolbox performing the augmentation/mid-level data fusion via ROI-based cube
    linking along the m/z dimension; run in MATLAB R2024a or newer
  repo: github.com/AdrianHaun/AriumMS
- name: ZMat
  role: Dependency used by AriumMS (ZMat toolbox by Qianqian Fang) for compression
    support in the MATLAB app
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:ca16aba6c879f5aed6f3b9e8bd1d2e17d46d8cc2189e238e0580756cc2eaac43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - ZMat
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mz-dimension-cube-linking

## Summary

AriumMS's mid-level data fusion (augmentation) step: links per-group data cubes (HILIC-MS, CE-MS) along the m/z dimension, labels features with the 'm/z @Time, Group' convention, and merges cross-group features sharing identical mass and occurrence count into one augmented feature table.

## When to use

Use when filtered per-group feature tables from different separation platforms (e.g., HILIC-MS ESI+/ESI- and CE-MS via the nanoCEasy interface) must be fused into a single augmented 'm/z @Time, Group' feature space for joint chemometric evaluation.

## When NOT to use

- Single-platform analysis: existing tools like XCMS Online, MetaboAnalyst, MetaboAnalystR, or MZmine suffice when only one separation technique is involved and no augmentation across methods is needed
- When precise autonomous linkage of features detected by two or more methods via MS/MS is required — MS/MS spectral networking is not implemented in AriumMS and is listed as future development
- Input is raw vendor MS data: files must first be converted to .mzXML or .mzML with msconvert (ProteoWizard) before the AriumMS pipeline, and augmentation operates downstream of ROI search, preprocessing, feature detection, and false-positive filtering

## Inputs

- Filtered per-group feature tables produced by the upstream AriumMS false-positive filter step (one data cube per separation group, e.g., HILIC-MS and CE-MS)
- User-defined group definitions assigning each dataset to its separation method (e.g., three CE-Orbitrap MS and two HILIC-Orbitrap MS methods)

## Outputs

- augmented_feature_table.csv — a group-labeled feature table spanning the combined m/z and time dimensions of all input groups, with features labeled 'm/z @Time, Group' and matched cross-group features (identical mass and occurrence count) merged

## How to apply

Load the false-positive-filtered per-group feature tables, treating each user-defined separation group (HILIC-MS, CE-MS) as a separate data cube. Apply the AriumMS augmentation routine, which uses the MSroiaug function developed by Tauler, Gorrochategui and Jaumot, to link the per-group cubes along the m/z dimension and combine m/z and time into a single fused dimension. Label every feature in the augmented space using the 'm/z @Time, Group' convention so features stay attributable to their originating separation method. Match features across groups that share an identical mass and identical occurrence count and collapse them into single augmented entries, completing the mid-level data fusion of the LC-MS + CE-MS tables. Write the result to augmented_feature_table.csv; the augmentation follows the false-positive filter and precedes chemometric data evaluation in the AriumMS pipeline (ROI search, preprocessing, feature detection and integration, false-positive filter, scaling and transformation, augmentation, chemometric evaluation).

## Related tools

- **AriumMS** (Toolbox performing the augmentation/mid-level data fusion via ROI-based cube linking along the m/z dimension; run in MATLAB R2024a or newer) — github.com/AdrianHaun/AriumMS
- **ZMat** (Dependency used by AriumMS (ZMat toolbox by Qianqian Fang) for compression support in the MATLAB app) — http://github.com/fangq/zmat

## Examples

```
% MATLAB (R2024a+): augmented = MSroiaug(filtered_feature_table); % links per-group cubes along m/z into the augmented 'm/z @Time, Group' space
```

## Evaluation signals

- Every row of augmented_feature_table.csv carries the 'm/z @Time, Group' label format
- Matched cross-group features with identical mass and identical occurrence count have been merged into single augmented entries
- The augmented table spans the combined m/z and time dimensions of all input groups (e.g., five combined datasets: three CE-Orbitrap MS and two HILIC-Orbitrap MS)
- Each feature remains attributable to its originating separation method (HILIC-MS vs. CE-MS) via the group label
- Mid-level fusion removes irrelevant information such as artifacts and noise from the entire LC-MS + CE-MS dataset before chemometric evaluation

## Limitations

- MS/MS spectral networking to precisely and autonomously connect features detected by two or more methods is not implemented; listed as future development
- Augmentation of spectroscopic data with chromatographic/electrophoretic data is not implemented; mentioned as possibly interesting future work
- Requires MATLAB R2024a or newer plus several toolboxes (Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, Database) for the app version
- No version number or changelog is stated for AriumMS in the article

## Evidence

- [discussion] AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts.: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS)"
- [abstract] AriumMS provides a novel mid-level data fusion approach for multi-platform data analysis to simplify and speed up multi-platform data processing and evaluation: "AriumMS provides a novel mid-level data fusion approach for multi-platform data analysis to simplify and speed up multi-platform data processing and evaluation"
- [readme] It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot https://doi.org/10.1038/protex.2015.102, and ZMat toolbox: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot https://doi.org/10.1038/protex.2015.102"
- [readme] For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project http://proteowizard.sourceforge.net/download.html"
