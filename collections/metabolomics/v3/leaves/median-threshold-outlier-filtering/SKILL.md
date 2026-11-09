---
name: median-threshold-outlier-filtering
description: Use when a CWT peak-detection step has produced ROI feature regions with
  intensity profiles from LC-MS/CE-MS metabolomics runs and noise features must be
  removed before feature integration and data augmentation.
when_to_use_negative:
- Input is raw vendor MS data that has not yet been ROI-searched and CWT peak-detected
  — the filter operates on detected peaks, not raw traces
- The analysis goal is filtering outliers across biological samples (e.g., removing
  bad injections) — this filter discriminates noise peaks from true peaks within a
  run, not sample-level outliers
- Single-platform analysis without downstream augmentation, where an existing single-platform
  tool (XCMS Online, MZmine, MetaboAnalyst) already provides adequate false-positive
  handling
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Implements the information-entropy false-positive peak filter as one step
    in its pipeline (ROI search, preprocessing, feature detection and integration,
    false-positive filter, scaling, transformation, augmentation, chemometric evaluation)
  repo: github.com/AdrianHaun/AriumMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:53f8284e7a065b6bc9ed3998b8ff5e5b5b48916ceaba7921267da5a7a4466a5a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# median-threshold-outlier-filtering

## Summary

An information-entropy false-positive peak filter (H = -p*log2(p) - q*log2(q)) that discards CWT-detected ROI peaks whose entropy exceeds the median entropy across all detected features, treating high-entropy (disordered) profiles as noise in AriumMS.

## When to use

Use when a CWT peak-detection step has produced ROI feature regions with intensity profiles from LC-MS/CE-MS metabolomics runs and noise features must be removed before feature integration and data augmentation.

## When NOT to use

- Input is raw vendor MS data that has not yet been ROI-searched and CWT peak-detected — the filter operates on detected peaks, not raw traces
- The analysis goal is filtering outliers across biological samples (e.g., removing bad injections) — this filter discriminates noise peaks from true peaks within a run, not sample-level outliers
- Single-platform analysis without downstream augmentation, where an existing single-platform tool (XCMS Online, MZmine, MetaboAnalyst) already provides adequate false-positive handling

## Inputs

- Peak table from upstream CWT peak detection containing detected feature regions-of-interest with intensity profiles
- Preprocessed centroided MS data converted to .mzXML or .mzML via ProteoWizard msconvert

## Outputs

- entropy_filtered_peaks.csv — peak table with median-entropy-flagged noise features removed
- Per-feature entropy H values (normal/variant point classification per Ju et al. scheme)

## How to apply

Load the peak table produced by upstream continuous wavelet transform (CWT) peak detection, which contains detected feature regions-of-interest with their intensity profiles. For each detected feature, classify the data points surrounding the peak maximum as 'normal' or 'variant' points per the Ju et al. scheme and compute the peak's information entropy H = -p*log2(p) - q*log2(q) from the normalized intensity profile. Compute the median entropy H across all detected features, then mark every feature with H greater than that median as a false-positive/noise feature — the median serves as the adaptive decision threshold instead of a fixed cutoff. Remove the flagged features from the peak table and write the filtered table to entropy_filtered_peaks.csv. The rationale is that genuine chromatographic/electrophoretic peaks (e.g., Gaussian-shaped CE-MS peaks) have ordered, low-entropy intensity profiles, whereas noise regions have high-entropy profiles; validate this by confirming synthetic Gaussian test peaks receive low H and survive while pure-noise test peaks receive high H and are discarded.

## Related tools

- **AriumMS** (Implements the information-entropy false-positive peak filter as one step in its pipeline (ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, transformation, augmentation, chemometric evaluation)) — github.com/AdrianHaun/AriumMS

## Evaluation signals

- Synthetic Gaussian-shaped test peaks receive low entropy and survive the filter, while pure-noise test peaks receive high entropy and are discarded
- All features in the output table have H less than or equal to the median entropy of the pre-filter peak table (no flagged features remain)
- Feature count of entropy_filtered_peaks.csv is strictly lower than the input CWT peak table, with the discarded set being those features above the median H
- The filtered table feeds cleanly into AriumMS's subsequent scaling, transformation, augmentation, and chemometric evaluation steps without artifact features propagating into LC-MS + CE-MS data fusion

## Limitations

- The median is a relative threshold: by construction, features with H above the median are always discarded regardless of the true noise content, so roughly half of detected features are flagged on every run
- Requires upstream CWT peak detection with intensity profiles; entropy cannot be computed on raw or unintegrated features
- AriumMS requires MATLAB R2024a or newer with Wavelet Toolbox (among other toolboxes) and no version number is stated for the software

## Evidence

- [discussion] Pipeline position of the filter: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] Filter purpose in multi-platform fusion: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset"
- [readme] Upstream file format and function dependencies: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
