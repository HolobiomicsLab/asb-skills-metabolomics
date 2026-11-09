---
name: entropy-based-peak-quality-filtering
description: Use when a feature table derived from CWT peak detection contains irregular,
  likely noise peaks, and you need a shape-based, parameter-light criterion to discard
  false positives before chemometric evaluation.
when_to_use_negative:
- The input is already a curated, filtered feature table — re-applying a median-entropy
  cutoff would discard genuine low-abundance features from the lower half of the entropy
  distribution
- The task is connecting features across methods by MS/MS spectral networking, which
  AriumMS does not implement (listed as future development), rather than removing
  noise peaks
- Peak shapes are expected to be highly non-Gaussian (e.g. broad or split CE peaks),
  where the Gaussian-shaped low-entropy assumption underlying the filter may misclassify
  real features as noise
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: AriumMS
  role: Implements the false-positive filter step after ROI search, preprocessing,
    and CWT feature detection; computes per-peak entropy and applies the median-entropy
    cutoff
  repo: github.com/AdrianHaun/AriumMS
- name: MATLAB (R2024a or newer)
  role: Runtime for the AriumMS app, including the Wavelet Toolbox used for the upstream
    CWT peak detection whose peak table the entropy filter consumes
- name: msconvert (ProteoWizard)
  role: Converts raw MS data to .mzXML or .mzML format as the upstream input to the
    AriumMS pipeline that produces the peak table filtered here
  repo: http://proteowizard.sourceforge.net/download.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:2a26d075d5f69e8b10f97d5c638512940281b4eca5bf358052bd0cc4aa23063f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - MATLAB (R2024a or newer)
  - msconvert (ProteoWizard)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# entropy-based-peak-quality-filtering

## Summary

A false-positive peak filter from the AriumMS toolbox that scores each CWT-detected peak by information entropy (H = -p*log2(p) - q*log2(q)) and discards peaks whose entropy exceeds the median entropy of all detected features, treating high-entropy (irregular) peak shapes as noise.

## When to use

Use when a feature table derived from CWT peak detection contains irregular, likely noise peaks, and you need a shape-based, parameter-light criterion to discard false positives before chemometric evaluation.

## When NOT to use

- The input is already a curated, filtered feature table — re-applying a median-entropy cutoff would discard genuine low-abundance features from the lower half of the entropy distribution
- The task is connecting features across methods by MS/MS spectral networking, which AriumMS does not implement (listed as future development), rather than removing noise peaks
- Peak shapes are expected to be highly non-Gaussian (e.g. broad or split CE peaks), where the Gaussian-shaped low-entropy assumption underlying the filter may misclassify real features as noise

## Inputs

- Peak table from AriumMS CWT peak-detection step: detected feature regions-of-interest with intensity profiles (from ROI search and preprocessing of .mzXML/.mzML data converted with msconvert)
- Raw MS data from multi-platform CE-MS and HILIC-MS runs (CE-Orbitrap MS and HILIC-Orbitrap MS methods) feeding the AriumMS ROIpeaks/MSroiaug pipeline

## Outputs

- Filtered peak table with high-entropy (noise) features removed, written to entropy_filtered_peaks.csv
- Per-feature entropy H values and the dataset-level median entropy used as the cutoff

## How to apply

Load the peak table produced by upstream CWT peak detection in AriumMS, which contains detected feature regions-of-interest with their intensity profiles. For each detected feature, classify the data points surrounding the peak maximum as 'normal' or 'variant' points per the Ju et al. scheme and compute the peak entropy H from the normalized intensity profile. Compute the median entropy H across all detected features and mark every feature with H greater than the median as a false-positive/noise feature, then remove the flagged features from the peak table. The rationale is that genuine chromatographic/electrophoretic peaks (e.g. Gaussian-shaped) concentrate intensity and receive low entropy, while pure-noise profiles distribute intensity irregularly and receive high entropy; the median provides an adaptive, dataset-relative cutoff. Write the surviving features to entropy_filtered_peaks.csv as the feature table entering downstream scaling, transformation, augmentation and chemometric evaluation.

## Related tools

- **AriumMS** (Implements the false-positive filter step after ROI search, preprocessing, and CWT feature detection; computes per-peak entropy and applies the median-entropy cutoff) — github.com/AdrianHaun/AriumMS
- **MATLAB (R2024a or newer)** (Runtime for the AriumMS app, including the Wavelet Toolbox used for the upstream CWT peak detection whose peak table the entropy filter consumes)
- **msconvert (ProteoWizard)** (Converts raw MS data to .mzXML or .mzML format as the upstream input to the AriumMS pipeline that produces the peak table filtered here) — http://proteowizard.sourceforge.net/download.html

## Evaluation signals

- Synthetic validation: Gaussian-shaped test peaks receive low entropy and survive the filter, while pure-noise test peaks receive high entropy and are discarded
- Output file entropy_filtered_peaks.csv exists and contains strictly fewer features than the input CWT peak table
- No retained feature has entropy H greater than the median entropy computed over the full pre-filter feature set
- Filtered feature table, after downstream scaling/transformation and augmentation, still separates known phenotypes (e.g. inhibitor-treated vs untreated Saccharomyces cerevisiae) in chemometric evaluation

## Limitations

- The median-entropy cutoff is relative to the dataset: if the majority of detected peaks are noise, the median shifts upward and real peaks may be discarded; if noise is rare, weak true peaks survive
- Point classification follows the Ju et al. 'normal'/'variant' scheme, so peaks whose shapes deviate from the assumed Gaussian-like template may be falsely flagged
- AriumMS does not implement MS/MS spectral networking or augmentation of spectroscopic data; the filter only removes shape-based noise and cannot validate feature identities
- No version number or explicit code/license availability statement was found in the article discussion; reproducibility depends on the GitHub repository (BSD-3-Clause per pre-resolved metadata)

## Evidence

- [discussion] False-positive filter position in AriumMS pipeline: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] Filter's role in removing noise from multi-platform data: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset"
- [readme] Underlying ROI functions and runtime environment used by AriumMS: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [readme] Toolchain, including Wavelet Toolbox for CWT, required to run the AriumMS app: "Required toolboxes for the app version: 
Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox, Image Processing Toolbox"
