---
name: chromatographic-peak-boundary-correction
description: Use when refining the start/stop limits of peaks detected in smoothed
  CE-MS or HILIC-MS ROI chromatograms (intensity-vs-retention-time traces), after
  CWT/Mexican-hat peak-center detection but before peak integration of RT, area, and
  height.
when_to_use_negative:
- Input is already a detected-peak/feature table (e.g. from MZmine or XCMS Online)
  rather than raw ROI traces — boundary correction would re-detect existing features
- The task requires MS/MS spectral networking to connect features across methods —
  AriumMS explicitly does not implement this (listed as future development)
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: Open-source MATLAB toolbox whose automatic peak detection on AriumMS-generated
    ROIs applies smoothing, noise estimation, CWT (Mexican hat) peak detection, friction-border
    + moving-std boundary correction, and peak integration
  repo: github.com/AdrianHaun/AriumMS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:1c14926acbbffe9843e037407385bf99292d8d5263a2ac07318e130c37c2e6d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chromatographic-peak-boundary-correction

## Summary

Two-step peak-boundary refinement used in the AriumMS evaluation pipeline for CE-MS/LC-MS region-of-interest traces: a friction-border rule followed by a moving standard deviation criterion that terminates boundaries where the signal rises above the estimated local noise floor.

## When to use

Use when refining the start/stop limits of peaks detected in smoothed CE-MS or HILIC-MS ROI chromatograms (intensity-vs-retention-time traces), after CWT/Mexican-hat peak-center detection but before peak integration of RT, area, and height.

## When NOT to use

- Input is already a detected-peak/feature table (e.g. from MZmine or XCMS Online) rather than raw ROI traces — boundary correction would re-detect existing features
- The task requires MS/MS spectral networking to connect features across methods — AriumMS explicitly does not implement this (listed as future development)

## Inputs

- Raw ROI chromatographic profiles (intensity-vs-retention-time traces) from the AriumMS repository test data
- Smoothed ROI signal (Savitzky-Golay-style smoothing) and its second derivative
- Local noise-level estimate (difference between smoothed and original signal)

## Outputs

- Peak-boundary-corrected peaks with apex retention time, area, and height
- detected_peaks.csv — one row per detected peak with non-null RT, area, and height fields

## How to apply

First smooth each raw ROI trace with a Savitzky-Golay-style smoother and estimate the local noise level as the difference between the smoothed and original signal, so that all peak criteria are scaled to the actual noise floor. Detect peak centers on the second derivative of the smoothed ROI via continuous wavelet transform with the Mexican hat mother wavelet, which suits the narrow peaks typical of CE-MS separations. Correct each peak's boundaries in two passes: apply the friction-border rule first, then refine the limits with a moving standard deviation criterion so that boundaries stop where the signal rises above local noise. Finally integrate each corrected peak to extract the apex retention time, area, and height, and assemble all peaks into a single table (e.g. detected_peaks.csv). This corresponds to the feature detection and integration stage of the AriumMS workflow (ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, transformation, augmentation).

## Related tools

- **AriumMS** (Open-source MATLAB toolbox whose automatic peak detection on AriumMS-generated ROIs applies smoothing, noise estimation, CWT (Mexican hat) peak detection, friction-border + moving-std boundary correction, and peak integration) — github.com/AdrianHaun/AriumMS

## Evaluation signals

- detected_peaks.csv exists and is parseable
- Every row of detected_peaks.csv carries non-null RT, area, and height fields
- Peak counts on the test ROIs are consistent with the expected detections for the AriumMS ROI search / feature detection and integration stage
- Corrected boundaries stop where the signal rises above the local noise level estimated from the smoothed-minus-raw difference
- Peak integration yields an apex retention time for every boundary-corrected peak (no peaks with boundaries but no apex)

## Limitations

- AriumMS requires MATLAB R2024a or newer and the Wavelet Toolbox (plus Bioinformatics, Statistics and Machine Learning, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes) for the app version
- CE-MS peaks are narrow by nature, so the Mexican hat CWT is tuned to that regime; peak characteristics of very different separation methods may need flexible per-method parameterization
- No version number or changelog is stated for AriumMS in the article, complicating reproducibility pinning

## Evidence

- [discussion] AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [discussion] narrow CE-MS peaks motivate the detection approach: "All CE-MS methods show in general narrow peaks and benefit from the sensitivity of the nanoCEasy interface."
- [readme] Wavelet Toolbox requirement for the AriumMS app: "Required toolboxes for the app version: Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox, Image Processing Toolbox, Signal Processing Toolbox"
