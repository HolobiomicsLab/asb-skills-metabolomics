---
name: untargeted-metabolomics-processing-parameterization
description: Use when raw CE-MS or LC-MS/HILIC-MS metabolomics acquisitions must be
  processed non-targetedly with AriumMS and per-method processing parameters (ROI
  thresholds, m/z error, alignment, occurrence filters) must be selected or DOE-optimized.
when_to_use_negative:
- The input is already a processed feature table — AriumMS handles raw converted data
  from ROI search onward, not post-processing of existing feature matrices
- Only a single separation platform is analyzed and no multi-platform augmentation
  is intended — existing single-platform tools such as XCMS Online, MetaboAnalyst,
  or MZmine may suffice
- MS/MS spectral networking or autonomous feature connection across methods is required
  — this is not implemented in AriumMS and is listed as future development
tools:
- name: AriumMS
  role: MATLAB toolbox executing ROI search, preprocessing, feature detection/integration,
    false-positive filtering, scaling, transformation, and augmentation with flexible
    per-method parameterization
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts vendor CE-MS/LC-MS raw files into the .mzXML or .mzML formats AriumMS
    consumes
  repo: http://proteowizard.sourceforge.net/download.html
- name: zmat
  role: ZMat compression toolbox (zlib/gzip/lzma/lz4/zstd/blosc2) used as a dependency
    by AriumMS for data handling
  repo: https://github.com/NeuroJSON/zmat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:efa97dc2e791a90004bdea9271e3c826f759a2d700be15cc223c9ab8789b62ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - msconvert (ProteoWizard)
  - zmat
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# untargeted-metabolomics-processing-parameterization

## Summary

Parameterization and D-optimal DOE-guided tuning of the AriumMS untargeted processing pipeline (ROI search, preprocessing, feature detection, false-positive filter) for multi-platform CE-MS and HILIC-MS metabolomics data, with per-method settings that respect the different peak characteristics of narrow CE peaks versus wider HILIC peaks.

## When to use

Use when raw CE-MS or LC-MS/HILIC-MS metabolomics acquisitions must be processed non-targetedly with AriumMS and per-method processing parameters (ROI thresholds, m/z error, alignment, occurrence filters) must be selected or DOE-optimized.

## When NOT to use

- The input is already a processed feature table — AriumMS handles raw converted data from ROI search onward, not post-processing of existing feature matrices
- Only a single separation platform is analyzed and no multi-platform augmentation is intended — existing single-platform tools such as XCMS Online, MetaboAnalyst, or MZmine may suffice
- MS/MS spectral networking or autonomous feature connection across methods is required — this is not implemented in AriumMS and is listed as future development

## Inputs

- Raw CE-MS and HILIC-MS metabolomics acquisitions (three CE-Orbitrap MS and two HILIC-Orbitrap MS methods) converted to .mzXML or .mzML
- Supplementary parameter documentation (Supplementary file1, DOCX) describing AriumMS non-target processing settings
- MATLAB R2024a (or newer) with required toolboxes for the AriumMS app

## Outputs

- Method-keyed JSON record of processing parameters (ROI intensity threshold, m/z error, mass spectra alignment quantile, minimum relative feature occurrence, minimum relative peak height, minimum ROI size) per analytical method
- AriumMS ROI search / preprocessing / false-positive filter outputs feeding feature detection, augmentation, and chemometric evaluation

## How to apply

Convert vendor MS files to .mzXML or .mzML with msconvert (ProteoWizard), since AriumMS consumes converted data in MATLAB (R2024a or newer with Bioinformatics, Statistics And Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes). In AriumMS, run ROI search, preprocessing, feature detection/integration, false-positive filter, scaling, and transformation per separation method in parallel. Set the m/z error to 0.01 Da, the ROI intensity threshold to 5–10% of the lowest base peak chromatogram/electropherogram intensity, minimum relative feature occurrence to 50%, and minimum relative peak height to 25%. Choose a per-method minimum ROI size of 15 for HILIC methods but a value below 10 for CE methods, reflecting CE's narrower peaks via the nanoCEasy interface. If parameters are unknown, use a D-optimal design of experiments to identify the significant parameters — the paper found ROI intensity threshold, mass spectra alignment, m/z error, and feature occurrence filter to be significant — and carry only those forward for optimization; record the final method-keyed settings as structured JSON.

## Related tools

- **AriumMS** (MATLAB toolbox executing ROI search, preprocessing, feature detection/integration, false-positive filtering, scaling, transformation, and augmentation with flexible per-method parameterization) — github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts vendor CE-MS/LC-MS raw files into the .mzXML or .mzML formats AriumMS consumes) — http://proteowizard.sourceforge.net/download.html
- **zmat** (ZMat compression toolbox (zlib/gzip/lzma/lz4/zstd/blosc2) used as a dependency by AriumMS for data handling) — https://github.com/NeuroJSON/zmat

## Examples

```
msconvert yeast_CE_sample.raw --mzML   # convert to .mzML before loading into the AriumMS MATLAB app (MATLAB R2024a or newer)
```

## Evaluation signals

- Every method entry in the output JSON carries all six parameter fields (ROI intensity threshold, m/z error, mass spectra alignment, minimum relative feature occurrence, minimum relative peak height, minimum ROI size)
- m/z error equals 0.01 Da for all methods; minimum relative feature occurrence is 50% and minimum relative peak height is 25%
- Minimum ROI size is 15 for HILIC methods but below 10 for the CE methods, consistent with CE's narrower peak characteristics
- Only the four DOE-significant parameters (ROI intensity threshold, mass spectra alignment, m/z error, feature occurrence filter) are carried forward for optimization
- ROI intensity threshold falls within the 5–10%-of-lowest-base-peak range documented for the five methods

## Limitations

- MS/MS spectral networking is not implemented; precisely connecting features detected by two or more methods is listed as future development
- No version number is stated for AriumMS, complicating exact software-state reproduction
- Definitive processing parameter values are only available in the electronic supplementary material (Supplementary file1, DOCX 1.51 MB), for which no direct access link is given in the main text
- Augmentation of spectroscopic data with chromatographic/electrophoretic data is not implemented

## Evidence

- [readme] README describes parameter-driven pipeline entry point: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation. It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [readme] README documents msconvert requirement for input formats: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
- [discussion] Pipeline steps executed by AriumMS: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [abstract] Per-method flexible parameterization rationale: "including parallel processing of datasets and flexible parameterization for processing of individual separation methods with different peak characteristics"
