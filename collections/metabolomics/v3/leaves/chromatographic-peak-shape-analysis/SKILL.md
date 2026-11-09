---
name: chromatographic-peak-shape-analysis
description: Use when raw CE-MS or LC-MS data (converted to .mzXML/.mzML via msconvert)
  has been peak-detected with the CWT/ROI search step and noise/false-positive peaks
  must be culled by shape before feature integration, scaling, and mid-level data
  fusion.
when_to_use_negative:
- Input is already a curated feature table (e.g. post-augmentation, post-scaling)
  — the filter operates on raw CWT/ROI peak tables before feature integration
- Dataset contains very few detected peaks — the median-entropy threshold is unstable
  without a large peak population to establish the median
- Peak shapes are not adequately described by a two-class normal/variant point scheme
  (e.g. heavily co-eluted or saturated peaks), where entropy no longer separates true
  peaks from noise
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: AriumMS
  role: Implements the CWT peak detection, the Ju et al. entropy-based false-positive
    filter, and the downstream scaling/transformation/augmentation pipeline as a MATLAB
    toolbox (R2024a or newer, requiring the Wavelet Toolbox for CWT)
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts vendor CE-MS/LC-MS raw files to .mzXML or .mzML format required upstream
    of ROI search and peak-shape filtering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:509f96035aaa291bad4259fd98da1c1a351a68434fe6a3816e038062321debeb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chromatographic-peak-shape-analysis

## Summary

A peak-shape-based false-positive filter from AriumMS that scores each CWT-detected CE-MS/LC-MS feature with its information entropy H = -p*log2(p) - q*log2(q) and discards features whose H exceeds the median entropy across all detected peaks, removing noise peaks before feature integration and augmentation.

## When to use

Use when raw CE-MS or LC-MS data (converted to .mzXML/.mzML via msconvert) has been peak-detected with the CWT/ROI search step and noise/false-positive peaks must be culled by shape before feature integration, scaling, and mid-level data fusion.

## When NOT to use

- Input is already a curated feature table (e.g. post-augmentation, post-scaling) — the filter operates on raw CWT/ROI peak tables before feature integration
- Dataset contains very few detected peaks — the median-entropy threshold is unstable without a large peak population to establish the median
- Peak shapes are not adequately described by a two-class normal/variant point scheme (e.g. heavily co-eluted or saturated peaks), where entropy no longer separates true peaks from noise

## Inputs

- Peak table from CWT peak detection with per-feature intensity profiles (ROI regions-of-interest)
- Converted MS files (.mzXML or .mzML, produced with ProteoWizard msconvert)
- Per-method processing parameters for CE-MS or HILIC-MS separations with different peak characteristics

## Outputs

- Filtered peak table (entropy_filtered_peaks.csv) with noise features removed
- Per-feature entropy H scores and the median-entropy threshold value

## How to apply

Load the peak table produced by the upstream CWT peak-detection step, containing detected feature regions-of-interest with their intensity profiles. For each detected peak, classify the data points surrounding the peak maximum as 'normal' or 'variant' points per the Ju et al. scheme, compute p and q as their fractions, and calculate the binary entropy H = -p*log2(p) - q*log2(q); a Gaussian-like peak concentrates points in one class and yields low H, while a noisy peak spreads them and yields high H. Compute the median entropy across all detected features and flag every feature with H greater than the median as a false-positive, then remove flagged features from the peak table and write the result (e.g. entropy_filtered_peaks.csv). In AriumMS this filter sits between 'feature detection and integration' and 'scaling, and transformation', feeding the subsequent augmentation and chemometric evaluation of multi-platform CE-MS/HILIC-MS datasets. The rationale is mid-level data fusion cleanup: removing irrelevant artifacts and noise early so that augmentation of CE-MS and HILIC-MS methods is not degraded.

## Related tools

- **AriumMS** (Implements the CWT peak detection, the Ju et al. entropy-based false-positive filter, and the downstream scaling/transformation/augmentation pipeline as a MATLAB toolbox (R2024a or newer, requiring the Wavelet Toolbox for CWT)) — github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts vendor CE-MS/LC-MS raw files to .mzXML or .mzML format required upstream of ROI search and peak-shape filtering)

## Evaluation signals

- Synthetic validation: Gaussian-shaped test peaks receive low entropy and survive the filter, while pure-noise test peaks receive high entropy and are discarded
- Flagging rule check: every removed feature has H strictly greater than the median entropy computed across all detected features in the run
- Entropy bound check: since p + q = 1 with two classes, each computed H must lie in [0, 1] (bits); H = 1 indicates a maximally ambiguous (noise-like) shape
- Row-count invariant: entropy_filtered_peaks.csv contains strictly fewer feature rows than the unfiltered CWT peak table, with no real (Gaussian-like) standard-mixture peaks removed
- Downstream sanity: mid-level data fusion of LC-MS + CE-MS yeast-extract datasets shows reduced artifacts and noise after filtering

## Limitations

- The median-entropy threshold is relative: if a run is dominated by noise peaks, the median shifts and genuine peaks may be misclassified
- MS/MS spectral networking is not implemented in AriumMS; connecting features across methods must be done externally
- Requires MATLAB R2024a or newer with specific toolboxes (Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox, Image Processing Toolbox, Signal Processing Toolbox, Parallel Computing Toolbox, Database Toolbox)
- No changelog or explicit code/license availability statement accompanies the toolbox description

## Evidence

- [discussion] AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset"
- [readme] Set parameters are then used to perform ROI search, data preprocessing and data augmentation.: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation."
- [other] computes the entropy H for each detected peak and considers peaks with H greater than the median entropy as noise: [evidence span withheld — anchor status: not_found]
- [other] Validate the filter on synthetic test peaks by confirming Gaussian-shaped test peaks receive low entropy and survive the filter: [evidence span withheld — anchor status: not_found]
- [readme] Required toolboxes for the app version: Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox: "Required toolboxes for the app version: Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox"
