---
name: multi-platform-data-augmentation
description: Use when you have feature tables from multiple separation methods (e.g.,
  HILIC-Orbitrap MS in ESI+ and ESI− plus CE-Orbitrap MS) that need to be combined
  into one augmented multi-platform dataset for joint target/suspect counting or chemometric
  evaluation.
when_to_use_negative:
- You only have a single separation platform (single-method data does not need augmentation;
  process it directly with the single-platform pipeline)
- You need MS/MS spectral networking to autonomously connect features detected by
  two or more methods — this is not implemented in AriumMS and is listed as future
  development
- Your input is already a fused/augmented feature table — augmentation would re-link
  already-merged entries
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: AriumMS
  role: Runs the augmentation routine (MSroiaug/ROIpeaks) that links per-group data
    cubes along the m/z dimension, applies the 'm/z @Time, Group' labeling, and merges
    cross-group features with identical mass and occurrence count into augmented_feature_table.csv
  repo: github.com/AdrianHaun/AriumMS
- name: ProteoWizard msconvert
  role: Converts vendor raw MS data to .mzXML or .mzML file format before processing
    in AriumMS
  repo: http://proteowizard.sourceforge.net/download.html
- name: fangq/zmat (ZMat toolbox)
  role: Compression toolbox dependency used by AriumMS
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_003
  - task_008
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:a0c553df34a7b63220a761965657c0c75f4a8954b890f04e04271f1f81756305
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - ProteoWizard msconvert
  - fangq/zmat (ZMat toolbox)
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# multi-platform-data-augmentation

## Summary

Mid-level data fusion of untargeted metabolomics feature tables from different separation platforms (e.g., HILIC-MS and CE-MS) using AriumMS's augmentation step, which links per-method data cubes along the m/z dimension so multi-platform data can be processed and evaluated as a single augmented dataset.

## When to use

Use when you have feature tables from multiple separation methods (e.g., HILIC-Orbitrap MS in ESI+ and ESI− plus CE-Orbitrap MS) that need to be combined into one augmented multi-platform dataset for joint target/suspect counting or chemometric evaluation.

## When NOT to use

- You only have a single separation platform (single-method data does not need augmentation; process it directly with the single-platform pipeline)
- You need MS/MS spectral networking to autonomously connect features detected by two or more methods — this is not implemented in AriumMS and is listed as future development
- Your input is already a fused/augmented feature table — augmentation would re-link already-merged entries

## Inputs

- Filtered per-group feature tables (one data cube per separation method, e.g., HILIC-MS and CE-MS groups) from upstream ROI search/preprocessing/false-positive filtering
- Raw MS measurements converted to .mzXML or .mzML (via ProteoWizard msconvert)
- User-defined groups, one per separation method

## Outputs

- augmented_feature_table.csv — augmented, group-labeled feature table spanning the combined m/z and time dimensions of all input groups
- Per-augmentation target/suspect counts (e.g., 24 targets / 38 suspects for ESI− augmentation; 19 targets / 31 suspects for ESI+ augmentation)

## How to apply

First run each separation group (per-method data cube) through AriumMS's upstream pipeline: ROI search, preprocessing, feature detection and integration, and method-specific false-positive filtering, since augmentation takes the filtered per-group feature tables as input. Then use the AriumMS augmentation routine (built on ROIpeaks/MSroiaug) to link the per-group data cubes along the m/z dimension, combining the m/z and time dimensions into a single fused dimension. Label every feature with the 'm/z @Time, Group' convention so features remain attributable to their originating separation method. Match features across groups that share an identical mass and identical occurrence count, collapsing them into single augmented entries (mid-level data fusion of the LC-MS + CE-MS tables). For the yeast extract use case, fuse the negative-ionization platforms (HILIC ESI−, CE alkaline BGE, CE acidic BGE) separately from the positive-ionization platforms (HILIC ESI+, CE cation), and write the augmented, group-labeled table to augmented_feature_table.csv.

## Related tools

- **AriumMS** (Runs the augmentation routine (MSroiaug/ROIpeaks) that links per-group data cubes along the m/z dimension, applies the 'm/z @Time, Group' labeling, and merges cross-group features with identical mass and occurrence count into augmented_feature_table.csv) — github.com/AdrianHaun/AriumMS
- **ProteoWizard msconvert** (Converts vendor raw MS data to .mzXML or .mzML file format before processing in AriumMS) — http://proteowizard.sourceforge.net/download.html
- **fangq/zmat (ZMat toolbox)** (Compression toolbox dependency used by AriumMS) — http://github.com/fangq/zmat

## Examples

```
msconvert yeast_HILIC_ESIneg.raw --mzML  # then load filtered per-group feature tables in AriumMS (MATLAB R2024a app) and run the augmentation step to write augmented_feature_table.csv
```

## Evaluation signals

- Every row of augmented_feature_table.csv carries the 'm/z @Time, Group' label format
- Cross-group features with identical mass and identical occurrence count are merged into single augmented entries
- The augmented table spans the combined m/z and time dimensions of all input separation groups
- For the yeast extract reference: ESI− augmentation (HILIC ESI− + CE alkaline BGE + CE acidic BGE) yields 24 targets / 38 suspects; ESI+ augmentation (HILIC ESI+ + CE cation) yields 19 targets / 31 suspects; single-method counts reproduce HILIC ESI+ 14 targets and CE cation 16 targets
- Single-method baselines (e.g., 14 targets for HILIC ESI+, 16 for CE cation) should be lower than the corresponding augmented counts, confirming fusion increased coverage

## Limitations

- MS/MS spectral networking for precisely and autonomously connecting features detected by two or more methods is not implemented; listed as future development
- Augmentation of spectroscopic data with chromatographic/electrophoretic data is not implemented; mentioned as possible future work
- AriumMS requires MATLAB R2024a or newer plus several toolboxes (Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, Database) for the app version
- Augmentation matches features on identical mass and identical occurrence count only; features that differ slightly across methods will not be merged

## Evidence

- [discussion] AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [readme] Augmentation links data cubes along m/z dimension: "All in one tool for untargeted Metabolomics by ROI and augmentation of multiple Data sets."
- [readme] Augmentation uses functions developed by Tauler et al.: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [discussion] Mid-level data fusion removes artifacts and noise from LC-MS + CE-MS dataset: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [intro] Existing tools do not focus on augmentation of different separation techniques: "However, the focus of these programs is not on the augmentation of different separation techniques."
