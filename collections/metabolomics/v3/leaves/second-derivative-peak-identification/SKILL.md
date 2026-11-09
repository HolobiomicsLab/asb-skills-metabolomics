---
name: second-derivative-peak-identification
description: Use when automatic peak picking is needed on raw intensity-vs-retention-time
  ROI traces (e.g. CE-MS or HILIC-MS profiles from AriumMS ROI search) where narrow
  peaks must be detected and integrated into an RT/area/height table.
when_to_use_negative:
- Input is already an integrated peak/feature table — this skill operates on raw ROI
  intensity-vs-RT traces before feature detection and integration
- Analytical peaks are broad LC-MS-style profiles rather than the narrow peaks typical
  of CE-MS, where the Mexican hat CWT is best suited
- The goal is downstream chemometrics (scaling, transformation, augmentation) rather
  than per-ROI peak detection
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AriumMS
  role: MATLAB toolbox (R2024a+); its AriumMSEval app performs smoothing, noise estimation,
    second-derivative CWT (Mexican hat) peak detection, boundary correction, and peak
    integration on test ROIs
  repo: https://github.com/AdrianHaun/AriumMS
- name: MATLAB Wavelet Toolbox
  role: Required dependency providing the continuous wavelet transform (CWT) with
    the Mexican hat mother wavelet used for peak center identification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:986a1ce230345cb6e3a22c3d9266e87b0e64851e9a31a3816a171b6f462c6dda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AdrianHaun/AriumMS
  tools:
  - AriumMS
  - MATLAB Wavelet Toolbox
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# second-derivative-peak-identification

## Summary

Automatic peak detection within chromatographic/electrophoretic region-of-interest traces by taking the second derivative of the smoothed signal and locating peak centers with continuous wavelet transform (CWT) using the Mexican hat mother wavelet, then integrating apex RT, area, and height. Suited to the narrow peaks typical of CE-MS separations, as implemented in the AriumMSEval evaluation app of the AriumMS toolbox.

## When to use

Use when automatic peak picking is needed on raw intensity-vs-retention-time ROI traces (e.g. CE-MS or HILIC-MS profiles from AriumMS ROI search) where narrow peaks must be detected and integrated into an RT/area/height table.

## When NOT to use

- Input is already an integrated peak/feature table — this skill operates on raw ROI intensity-vs-RT traces before feature detection and integration
- Analytical peaks are broad LC-MS-style profiles rather than the narrow peaks typical of CE-MS, where the Mexican hat CWT is best suited
- The goal is downstream chemometrics (scaling, transformation, augmentation) rather than per-ROI peak detection

## Inputs

- Raw intensity-vs-retention-time ROI chromatographic profiles (from AriumMS ROI search on .mzXML/.mzML CE-MS or HILIC-MS data)
- Smoothing and noise-estimation parameters for the ROI traces

## Outputs

- detected_peaks.csv — peak table with non-null retention time (apex), area, and height per detected peak
- Per-peak boundary positions after friction-border and moving-std correction

## How to apply

For each ROI chromatogram, apply Savitzky-Golay-style smoothing and estimate the local noise level as the difference between the smoothed and original signal so peak criteria scale to the actual noise floor. Compute the second derivative of the smoothed trace and identify candidate peak centers via continuous wavelet transform using the Mexican hat mother wavelet, which matches the narrow peaks typical of CE-MS separations. Correct peak boundaries in two steps: first the friction-border rule, then refinement with a moving standard deviation criterion so boundaries stop where the signal rises above local noise. Integrate each detected peak to extract retention time (apex), area, and height, and assemble all peaks into a single table (detected_peaks.csv).

## Related tools

- **AriumMS** (MATLAB toolbox (R2024a+); its AriumMSEval app performs smoothing, noise estimation, second-derivative CWT (Mexican hat) peak detection, boundary correction, and peak integration on test ROIs) — https://github.com/AdrianHaun/AriumMS
- **MATLAB Wavelet Toolbox** (Required dependency providing the continuous wavelet transform (CWT) with the Mexican hat mother wavelet used for peak center identification)

## Evaluation signals

- detected_peaks.csv exists and is parseable
- Every row of detected_peaks.csv carries non-null RT, area, and height fields
- Peak counts on the test ROIs are consistent with the expected detections for the AriumMS ROI search / feature detection and integration stage
- Peak boundaries stop where the signal rises above the local noise (moving standard deviation criterion), confirming the friction-border + moving-std correction ran
- Noise-scaled criteria: smoothing-vs-original difference was used to set thresholds, so detections track the actual noise floor of each ROI

## Limitations

- Mexican hat CWT is tuned to narrow CE-MS peaks; peak shapes from other separation techniques may require different peak characteristics/parameterization
- Requires MATLAB R2024a or newer with Wavelet Toolbox (plus Bioinformatics, Statistics and Machine Learning, Image Processing, Signal Processing, Parallel Computing, and Database Toolboxes) for the app version
- No version number is stated for AriumMS and no changelog is available, complicating reproducibility of detection behavior

## Evidence

- [other] CWT Mexican hat identification: [evidence span withheld — anchor status: not_found]
- [discussion] Feature detection within AriumMS pipeline: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [readme] Wavelet Toolbox dependency for CWT: "Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox, Image Processing Toolbox, Signal Processing Toolbox, Parallel Computing Toolbox, Database Toolbox"
- [discussion] Narrow CE-MS peaks motivate the method: "All CE-MS methods show in general narrow peaks and benefit from the sensitivity of the nanoCEasy interface."
