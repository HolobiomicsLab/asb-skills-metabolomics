---
name: false-positive-feature-discrimination
description: Use when a peak table from CWT peak detection (e.g., AriumMS CE-MS or
  HILIC-MS runs) contains suspected false-positive features and a shape-based, parameter-light
  noise criterion is needed before augmentation or mid-level data fusion.
when_to_use_negative:
- The input is already a curated feature table that has passed a false-positive filter
  — re-applying the median-entropy cutoff would arbitrarily discard another ~half
  of the features.
- You need a blank-control or intensity-threshold-based noise removal rather than
  a peak-shape criterion, since this filter judges only the disorder of the intensity
  profile around the peak maximum.
- Features come from a detector workflow that does not provide the per-point intensity
  profile around the peak maximum required for the Ju et al. normal/variant point
  classification.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Implements the false-positive filter step (via the information-entropy peak
    filter after CWT peak detection) within its ROI search → preprocessing → feature
    detection pipeline
  repo: https://github.com/AdrianHaun/AriumMS
- name: Wavelet Toolbox
  role: MATLAB toolbox required by the AriumMS app to perform the upstream CWT peak
    detection whose peak table feeds the entropy filter
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:4f8a667f407e3aa7799a7b0df427e5f15e688960594c1c036c051dc11388bfc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AdrianHaun/AriumMS
  tools:
  - AriumMS
  - Wavelet Toolbox
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# false-positive-feature-discrimination

## Summary

Information-entropy false-positive peak filter from AriumMS that discards CWT-detected features whose peak entropy H exceeds the median entropy of all detected peaks, treating high-entropy (disordered) intensity profiles as noise rather than true chromatographic/electrophoretic peaks.

## When to use

Use when a peak table from CWT peak detection (e.g., AriumMS CE-MS or HILIC-MS runs) contains suspected false-positive features and a shape-based, parameter-light noise criterion is needed before augmentation or mid-level data fusion.

## When NOT to use

- The input is already a curated feature table that has passed a false-positive filter — re-applying the median-entropy cutoff would arbitrarily discard another ~half of the features.
- You need a blank-control or intensity-threshold-based noise removal rather than a peak-shape criterion, since this filter judges only the disorder of the intensity profile around the peak maximum.
- Features come from a detector workflow that does not provide the per-point intensity profile around the peak maximum required for the Ju et al. normal/variant point classification.

## Inputs

- Peak table from CWT peak detection with per-feature intensity profiles (AriumMS ROI search / feature detection output)
- Detected feature regions-of-interest from multi-platform CE-MS and HILIC-MS datasets (mzXML/mzML converted via msconvert)

## Outputs

- entropy_filtered_peaks.csv — filtered feature table with high-entropy (H > median H) false-positive features removed
- Per-feature entropy values H for the detected peaks

## How to apply

Load the peak table produced by the upstream continuous wavelet transform (CWT) peak-detection step, containing detected feature regions-of-interest with their intensity profiles. For each detected feature, classify the data points surrounding the peak maximum as 'normal' or 'variant' points per the Ju et al. scheme and compute the entropy H = -p*log2(p) - q*log2(q) from the normalized intensity profile. Compute the median entropy H across all detected features and flag every feature with H greater than the median as a false-positive/noise feature. Remove flagged features and write the result to entropy_filtered_peaks.csv. The rationale is that true, sharp CE-MS/HILIC-MS peaks concentrate intensity in few points (low entropy), whereas noise profiles spread intensity irregularly (high entropy).

## Related tools

- **AriumMS** (Implements the false-positive filter step (via the information-entropy peak filter after CWT peak detection) within its ROI search → preprocessing → feature detection pipeline) — https://github.com/AdrianHaun/AriumMS
- **Wavelet Toolbox** (MATLAB toolbox required by the AriumMS app to perform the upstream CWT peak detection whose peak table feeds the entropy filter)

## Evaluation signals

- Synthetic Gaussian-shaped test peaks receive low entropy H and survive the filter, while pure-noise test peaks receive high entropy H and are discarded.
- Every retained feature has H ≤ the median entropy of the unfiltered peak table; every removed feature has H > median.
- The filtered output is written as entropy_filtered_peaks.csv and contains strictly fewer features than the input CWT peak table.
- Sharp, narrow CE-MS peaks (a stated strength of the nanoCEasy interface) are retained rather than removed, indicating the filter is not over-pruning true features.
- Downstream mid-level data fusion on the filtered table removes artifacts and noise from the combined LC-MS + CE-MS dataset without eliminating metabolome-level group separation (e.g., treated vs. untreated S. cerevisiae).

## Limitations

- The H > median-entropy threshold is relative: it removes roughly the upper half of detected features by construction, so its behavior depends on the noise/content composition of each dataset.
- The entropy computation depends on the Ju et al. normal/variant point classification around the peak maximum, so poorly integrated or shouldered peaks may receive misleading entropy values.
- No MS/MS spectral confirmation is integrated (listed only as future development in AriumMS), so feature identity cannot be used to cross-check filtered features.

## Evidence

- [discussion] AriumMS pipeline stages including the filter: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] Filter purpose in mid-level data fusion: "AriumMS was successfully applied for mid-level data fusion to remove irrelevant information such as artifacts and noise from the entire analytical dataset"
- [other] Entropy filter criterion (source card finding): [evidence span withheld — anchor status: not_found]
- [readme] Upstream ROI/CWT functions used by AriumMS: "It uses functions (ROIpeaks, MSroiaug) developed by Romà Tauler, Eva Gorrochategui and Joaquim Jaumot"
- [readme] Required MATLAB toolboxes for the app including CWT capability: "Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox, Image Processing Toolbox, Signal Processing Toolbox, Parallel Computing Toolbox"
