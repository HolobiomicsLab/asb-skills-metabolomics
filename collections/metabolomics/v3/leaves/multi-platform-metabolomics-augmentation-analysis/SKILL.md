---
name: multi-platform-metabolomics-augmentation-analysis
description: Use when raw metabolomics data were acquired across multiple separation
  platforms (e.g., HILIC-MS plus CE-MS with different BGEs/ionization modes) and you
  need one augmented feature table and regulation-call matrix instead of separate
  per-platform results.
when_to_use_negative:
- You have a single-platform dataset only — AriumMS's value lies in augmentation of
  different separation techniques; standard single-platform tools (XCMS Online, MetaboAnalyst,
  MZmine) suffice and existing programs are not designed for augmenting different
  techniques
- The input is already a fused/augmented feature table — proceed directly to fold-change
  and regulation-call computation instead of re-running ROI search and augmentation
- You need MS/MS spectral networking or augmentation of spectroscopic data — neither
  is implemented in AriumMS and both are listed as future developments
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AriumMS
  role: Primary toolbox performing ROI search, preprocessing, feature detection, false-positive
    filtering, scaling/transformation, augmentation, and chemometric evaluation of
    multi-platform data
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts vendor MS data to .mzXML or .mzML format required as AriumMS input
  repo: http://proteowizard.sourceforge.net/download.html
- name: zmat (fangq/zmat)
  role: Zlib/gzip/lzma/lz4/zstd/blosc2 compression toolbox used by AriumMS for data
    handling
  repo: http://github.com/fangq/zmat
- name: nanoCEasy interface
  role: CE-MS coupling delivering narrow peaks and sensitivity for the three CE-Orbitrap
    MS methods included in the augmentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:6c4f1048e4b4afa7bc2b6c95a2e094ba36c386ef913094371beb19c6759e71d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - msconvert (ProteoWizard)
  - nanoCEasy interface
  - zmat (fangq/zmat)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multi-platform-metabolomics-augmentation-analysis

## Summary

Augments and jointly evaluates multi-platform untargeted metabolomics datasets (three CE-Orbitrap MS and two HILIC-Orbitrap MS methods) using AriumMS's region-of-interest algorithm and mid-level data fusion, enabling single chemometric evaluation across ESI positive and ESI negative separations.

## When to use

Use when raw metabolomics data were acquired across multiple separation platforms (e.g., HILIC-MS plus CE-MS with different BGEs/ionization modes) and you need one augmented feature table and regulation-call matrix instead of separate per-platform results.

## When NOT to use

- You have a single-platform dataset only — AriumMS's value lies in augmentation of different separation techniques; standard single-platform tools (XCMS Online, MetaboAnalyst, MZmine) suffice and existing programs are not designed for augmenting different techniques
- The input is already a fused/augmented feature table — proceed directly to fold-change and regulation-call computation instead of re-running ROI search and augmentation
- You need MS/MS spectral networking or augmentation of spectroscopic data — neither is implemented in AriumMS and both are listed as future developments

## Inputs

- Raw LC-MS/CE-MS acquisition files converted to mzXML or mzML
- Five dataset groups: three CE-Orbitrap MS methods (nanoCEasy interface; alkaline BGE, acidic BGE, cationic) and two HILIC-Orbitrap MS methods (ESI positive, ESI negative)
- AriumMS processing parameters (per-separation-method settings for peak characteristics)
- Sample metadata distinguishing treated vs. control Saccharomyces cerevisiae extracts and target metabolite list (e.g., 40 targets)

## Outputs

- Augmented feature intensity table per augmentation group (mid-level fused across HILIC and CE methods)
- Chemometric evaluation results (e.g., PCA/phenotype separation of yeast metabolomes)
- Regulation-call matrix (down/stable/up/knock out/induced/n.a.) for each target metabolite per augmentation group
- Detected-target counts per augmentation group (e.g., 24 for ESI negative, 19 for ESI positive)

## How to apply

Convert vendor raw files to mzXML or mzML with ProteoWizard's msconvert before loading into AriumMS (a MATLAB R2024a+ app requiring Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes). Group datasets by ionization mode — e.g., the ESI negative augmentation (HILIC [ESI−], CE alkaline BGE, CE acidic BGE) and the ESI positive augmentation (HILIC [ESI+], CE cation) — and run the AriumMS pipeline: ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation, followed by augmentation and chemometric evaluation, with parallel processing and per-method flexible parameterization. The augmentation relies on the ROIpeaks/MSroiaug functions (Tauler/Gorrochategui/Jaumot protocol) and ZMat compression (fangq/zmat). For phenotype comparison, compute per-metabolite fold changes between treated (growth-inhibitor) and control Saccharomyces cerevisiae samples on the mid-level fused feature tables and assign regulation calls (down/stable/up/knock out/induced, n.a. if undetected), reporting detected-target counts per augmentation.

## Related tools

- **AriumMS** (Primary toolbox performing ROI search, preprocessing, feature detection, false-positive filtering, scaling/transformation, augmentation, and chemometric evaluation of multi-platform data) — github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts vendor MS data to .mzXML or .mzML format required as AriumMS input) — http://proteowizard.sourceforge.net/download.html
- **zmat (fangq/zmat)** (Zlib/gzip/lzma/lz4/zstd/blosc2 compression toolbox used by AriumMS for data handling) — http://github.com/fangq/zmat
- **nanoCEasy interface** (CE-MS coupling delivering narrow peaks and sensitivity for the three CE-Orbitrap MS methods included in the augmentation)

## Examples

```
msconvert yeast_extract_*.raw --mzML  # then load the mzML files into the AriumMS MATLAB app and run ROIpeaks/MSroiaug per augmentation group
```

## Evaluation signals

- The ESI negative augmentation (HILIC [ESI−], CE alkaline, CE acidic) detects 24 of the 40 target metabolites; the ESI positive augmentation (HILIC [ESI+], CE cation) detects 19
- Regulation calls are assignable for every target in each augmentation group, with 'n.a.' used only where a target is undetected in that group
- Mid-level fused data separate treated vs. control Saccharomyces cerevisiae metabolomes (AriumMS successfully differentiated yeast treated with a growth inhibitor)
- Fused dataset is free of artifacts/noise: AriumMS mid-level fusion removes irrelevant information such as artifacts and noise from the combined LC-MS + CE-MS dataset
- All input files are valid mzXML/mzML conversions and processing completed through the full ROI-to-augmentation chain without per-platform manual merging

## Limitations

- MS/MS spectral networking is not implemented; it is listed as future development for precisely connecting features detected by two or more methods
- Augmentation of spectroscopic data with chromatographic/electrophoretic data is not implemented (possible future work)
- Requires MATLAB R2024a or newer plus seven specific toolboxes for the app version; no AriumMS version number is stated and no changelog exists
- AriumMS handles untargeted ROI/augmentation processing but does not itself assign metabolite regulation calls — fold changes and down/stable/up/knock out/induced labeling must be computed downstream from the fused intensity tables

## Evidence

- [abstract] AriumMS offers augmented data analysis of several separation techniques utilizing a region-of-interest algorithm: "AriumMS offers augmented data analysis of several separation techniques utilizing a region-of-interest algorithm"
- [abstract] Five combined datasets: "To demonstrate the capabilities of AriumMS, five datasets were combined"
- [discussion] Mid-level fusion removes artifacts and noise: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset (LC-MS + CE-MS) of yeast extracts."
- [discussion] Pipeline order: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [readme] File conversion requirement: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
- [readme] Underlying ROI functions and MATLAB version: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [intro] Existing tools not built for augmentation: "However, the focus of these programs is not on the augmentation of different separation techniques."
