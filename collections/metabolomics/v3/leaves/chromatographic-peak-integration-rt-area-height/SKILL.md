---
name: chromatographic-peak-integration-rt-area-height
description: Use when untargeted metabolomics ROIs (intensity-vs-retention-time traces
  from CE-MS or HILIC-MS runs, e.g. .mzML/.mzXML converted profiles) must be converted
  into a per-peak table of apex RT, area, and height for feature integration.
when_to_use_negative:
- Input is already a feature table or peak list with RT/area/height values — peak
  detection and integration would duplicate existing results
- Peaks are broad and Gaussian-like rather than the narrow peaks typical of CE-MS
  separations, where a Mexican-hat CWT approach may be unnecessarily specialized
- The goal is ROI finding or augmentation/chemometric evaluation rather than converting
  ROIs into integrated peak quantities
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: AriumMS
  role: MATLAB app performing ROI search, preprocessing, and feature detection and
    integration, including the CWT (Mexican hat) peak detection with friction-border
    and moving-std boundary correction and peak integration into a peak table
  repo: github.com/AdrianHaun/AriumMS
- name: MATLAB Wavelet Toolbox
  role: Required toolbox providing the continuous wavelet transform (Mexican hat)
    functionality used for candidate peak-center detection in the app version of AriumMS
- name: msconvert (ProteoWizard)
  role: Upstream conversion of vendor MS data to .mzXML or .mzML file format expected
    by the AriumMS pipeline before ROI-based peak integration
  repo: http://proteowizard.sourceforge.net/download.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:466326c1286f133e7eeae5d589944604cd060ef65300a1eddaea499ccd886b0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - MATLAB Wavelet Toolbox
  - msconvert (ProteoWizard)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chromatographic-peak-integration-rt-area-height

## Summary

Integrates peaks detected within chromatographic/electrophoretic regions of interest (ROIs) to extract apex retention time, area, and height, as implemented in the AriumMS feature-detection stage for multi-platform CE-MS/LC-MS metabolomics data. It turns raw intensity-vs-retention-time ROI traces into a structured peak table usable for downstream augmentation and chemometrics.

## When to use

Use when untargeted metabolomics ROIs (intensity-vs-retention-time traces from CE-MS or HILIC-MS runs, e.g. .mzML/.mzXML converted profiles) must be converted into a per-peak table of apex RT, area, and height for feature integration.

## When NOT to use

- Input is already a feature table or peak list with RT/area/height values — peak detection and integration would duplicate existing results
- Peaks are broad and Gaussian-like rather than the narrow peaks typical of CE-MS separations, where a Mexican-hat CWT approach may be unnecessarily specialized
- The goal is ROI finding or augmentation/chemometric evaluation rather than converting ROIs into integrated peak quantities

## Inputs

- Raw intensity-vs-retention-time chromatographic/electrophoretic ROI profiles from AriumMS ROI search (MS data previously converted to .mzXML or .mzML with ProteoWizard msconvert)
- Smoothed ROI signals with associated local noise estimates (smoothed-minus-original difference)
- Peak-detection parameters (smoothing settings, wavelet scale, noise-scaling criteria) configured in the AriumMS app

## Outputs

- Detected-peak table (detected_peaks.csv) with one row per peak and non-null RT (apex), area, and height fields
- Per-peak boundary annotations refined by friction-border and moving-standard-deviation criteria

## How to apply

First smooth each ROI (Savitzky-Golay-style) and estimate the local noise level from the difference between the smoothed and original signal, so that peak criteria scale to the actual noise floor. Compute the second derivative of the smoothed ROI and find candidate peak centers by continuous wavelet transform (CWT) using the Mexican hat mother wavelet, which suits the narrow peaks typical of CE-MS separations. Correct peak boundaries in two steps: apply the friction-border rule, then refine limits with a moving standard deviation criterion so boundaries stop where the signal rises above local noise. Finally integrate each detected peak to extract retention time (apex), area, and height, and assemble all peaks into a single table (e.g. detected_peaks.csv). In AriumMS this is the 'feature detection and integration' stage that follows ROI search and preprocessing.

## Related tools

- **AriumMS** (MATLAB app performing ROI search, preprocessing, and feature detection and integration, including the CWT (Mexican hat) peak detection with friction-border and moving-std boundary correction and peak integration into a peak table) — github.com/AdrianHaun/AriumMS
- **MATLAB Wavelet Toolbox** (Required toolbox providing the continuous wavelet transform (Mexican hat) functionality used for candidate peak-center detection in the app version of AriumMS)
- **msconvert (ProteoWizard)** (Upstream conversion of vendor MS data to .mzXML or .mzML file format expected by the AriumMS pipeline before ROI-based peak integration) — http://proteowizard.sourceforge.net/download.html

## Evaluation signals

- detected_peaks.csv exists and is parseable
- Every row of detected_peaks.csv carries non-null RT, area, and height fields
- Peak counts on test ROIs are consistent with the expected detections for the AriumMS ROI search / feature detection and integration stage
- Peak boundaries terminate where the signal rises above the local noise floor per the moving-standard-deviation criterion
- Peak criteria are scaled to the actual noise floor, i.e. noise estimated from smoothed-vs-original signal differences

## Limitations

- AriumMS requires MATLAB R2024a or newer and, for the app version, the Wavelet Toolbox (plus Bioinformatics, Statistics and Machine Learning, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes)
- MS/MS spectral networking to precisely connect features across methods is not implemented; listed as a future development
- No version number is stated for AriumMS and no changelog is available, complicating reproducibility of detection settings

## Evidence

- [discussion] Pipeline scope of AriumMS including the feature detection and integration stage this skill covers: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation"
- [other] CWT Mexican hat peak-center detection on the second derivative of the smoothed ROI: [evidence span withheld — anchor status: not_found]
- [other] Two-step boundary correction and peak integration to apex RT, area, and height: [evidence span withheld — anchor status: not_found]
- [readme] Wavelet Toolbox requirement underpinning the CWT-based detection in the app: "Required toolboxes for the app version: 
Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox, Image Processing Toolbox, Signal Processing Toolbox"
- [readme] Parameter-driven ROI processing chain feeding the peak integration stage: "Set parameters are then used to perform ROI search, data preprocessing and data augmentation."
