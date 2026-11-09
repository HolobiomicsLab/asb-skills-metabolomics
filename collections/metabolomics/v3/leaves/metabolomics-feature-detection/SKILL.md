---
name: metabolomics-feature-detection
description: Use when raw Orbitrap MS files from CE-MS (nanoCEasy interface) or HILIC-MS
  separations in ESI+ / ESI− mode must be converted to features — ROI search, preprocessing,
  integration, and false-positive filtering — especially for later multi-platform
  augmentation.
when_to_use_negative:
- Input is already a processed feature table — ROI search, preprocessing, and feature
  detection are unnecessary and would not apply
- The goal is MS/MS spectral networking or autonomous feature linking across methods
  — this is not implemented in AriumMS (listed as future development)
- Single-platform data where standard single-platform tools (XCMS Online, MetaboAnalyst,
  MZmine) suffice and no augmentation of different separation techniques is intended
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AriumMS
  role: Main toolbox performing ROI search, preprocessing, feature detection/integration,
    false-positive filtering, and augmentation of multi-platform CE-MS/HILIC-MS datasets
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts vendor raw MS data to .mzXML or .mzML format required as AriumMS
    input
  repo: http://proteowizard.sourceforge.net/download.html
- name: zmat
  role: ZMat compression toolbox dependency used by AriumMS (functions by Qianqian
    Fang)
  repo: github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:6eaab1fb4c34aa9de06e8b5746168d993b550425df48af3193407bcac7a0b989
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - msconvert (ProteoWizard)
  - zmat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-feature-detection

## Summary

Detect and integrate untargeted metabolomics features from multi-platform CE-MS and HILIC-MS raw data using the AriumMS ROI-based pipeline (ROI search, preprocessing, feature detection/integration, false-positive filtering), with method-specific parameters per separation technique.

## When to use

Use when raw Orbitrap MS files from CE-MS (nanoCEasy interface) or HILIC-MS separations in ESI+ / ESI− mode must be converted to features — ROI search, preprocessing, integration, and false-positive filtering — especially for later multi-platform augmentation.

## When NOT to use

- Input is already a processed feature table — ROI search, preprocessing, and feature detection are unnecessary and would not apply
- The goal is MS/MS spectral networking or autonomous feature linking across methods — this is not implemented in AriumMS (listed as future development)
- Single-platform data where standard single-platform tools (XCMS Online, MetaboAnalyst, MZmine) suffice and no augmentation of different separation techniques is intended

## Inputs

- Raw Orbitrap MS measurement files from CE-MS and HILIC-MS runs (converted to .mzXML or .mzML via msconvert)
- Metabolite standard mixtures and biological extracts (e.g., Saccharomyces cerevisiae yeast extracts) as sample batches
- Target/suspect metabolite lists for coverage counting

## Outputs

- Per-method feature tables (detected and integrated ROI features)
- Per-method target and suspect metabolite counts (e.g., 14 HILIC ESI+ targets; 16 CE cation targets)
- Augmented (mid-level fused) feature matrices per ionization polarity (e.g., ESI−: 24 targets/38 suspects; ESI+: 19 targets/31 suspects)

## How to apply

First convert vendor raw files to .mzXML or .mzML with msconvert (ProteoWizard), since AriumMS consumes converted MS data. Load the datasets (e.g., three CE-Orbitrap MS and two HILIC-Orbitrap MS methods on yeast extracts) into AriumMS under MATLAB R2024a or newer and run the pipeline of ROI search, preprocessing, feature detection and integration, false-positive filtering, scaling and transformation. Apply flexible, method-specific parameterization per separation method — CE-MS peaks are narrow while HILIC-MS peaks differ in shape, so each method needs its own processing settings (parallel batch processing is supported). First process each single method separately (e.g., HILIC ESI+ and CE cation) to obtain per-method feature and target counts, then optionally perform mid-level data fusion (augmentation, MSroiaug/ROIpeaks functions) grouping by ionization polarity to remove artifacts and noise before chemometric evaluation. Compare resulting target/suspect counts against reference values to validate the processing settings.

## Related tools

- **AriumMS** (Main toolbox performing ROI search, preprocessing, feature detection/integration, false-positive filtering, and augmentation of multi-platform CE-MS/HILIC-MS datasets) — github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts vendor raw MS data to .mzXML or .mzML format required as AriumMS input) — http://proteowizard.sourceforge.net/download.html
- **zmat** (ZMat compression toolbox dependency used by AriumMS (functions by Qianqian Fang)) — github.com/fangq/zmat

## Examples

```
msconvert --mzML *.raw
```

## Evaluation signals

- Single-method target counts match reported references: 14 targets for HILIC ESI+ and 16 targets for CE cation
- ESI− augmentation (HILIC ESI− + CE alkaline BGE + CE acidic BGE) reproduces 24 targets / 38 suspects
- ESI+ augmentation (HILIC ESI+ + CE cation) reproduces 19 targets / 31 suspects
- Deviations from these reference counts indicate incorrect processing or false-positive filter settings and require re-tuning of method-specific parameters

## Limitations

- No universal parameter set exists: each separation method requires its own preprocessing parameters because peak characteristics differ (narrow CE peaks vs. LC peaks)
- MS/MS spectral networking for connecting features across methods is not implemented (future work)
- Requires MATLAB R2024a or newer plus specific toolboxes (Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, Database) for the app version
- No version number or changelog is stated for AriumMS

## Evidence

- [discussion] Pipeline stages supported by article: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [abstract] ROI algorithm and multi-method support: "AriumMS offers augmented data analysis of several separation techniques utilizing a region-of-interest algorithm"
- [abstract] Method-specific parameterization rationale: "including parallel processing of datasets and flexible parameterization for processing of individual separation methods with different peak characteristics"
- [readme] Raw-file conversion requirement: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
- [discussion] Mid-level fusion purpose: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
