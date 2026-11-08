---
name: wavelet-based-peak-detection-mexican-hat
description: Use when you have ROI chromatographic profiles (intensity vs. retention
  time) extracted from CE-MS or LC-MS data by the AriumMS ROI search, and need automatic
  peak detection with integration of apex RT, area, and height — especially for the
  narrow peaks typical of CE-MS separations.
when_to_use_negative:
- The full mzML/mzXML runs have not yet been reduced to regions of interest by the
  AriumMS ROI search — this skill operates on ROI traces, not raw files
- Input is already a detected-peak/feature table needing augmentation, false-positive
  filtering, or chemometric evaluation rather than peak detection
- You need MS/MS spectral networking to connect features across methods — explicitly
  listed as future development, not part of this workflow
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3652
tools:
- name: AriumMS
  role: Host app whose ROI search generates the ROI chromatographic profiles; the
    evaluation app (AriumMSEval) performs the CWT/Mexican-hat peak detection on those
    ROIs
  repo: github.com/AdrianHaun/AriumMS
- name: MATLAB Wavelet Toolbox
  role: Required toolbox providing the continuous wavelet transform with the Mexican
    hat mother wavelet used for peak detection in the AriumMS app version
- name: zmat
  role: Compression/decompression toolbox bundled as a dependency of the AriumMS MATLAB
    codebase
  repo: http://github.com/fangq/zmat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:c1f87c32245fd5d551e887f2f0f890db3de0e20de1fbaefd922f056e1cdab891
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - MATLAB Wavelet Toolbox
  - zmat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3652
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# wavelet-based-peak-detection-mexican-hat

## Summary

Automatic peak detection and integration for CE-MS/LC-MS regions of interest in the AriumMSEval workflow: candidate peaks are found by continuous wavelet transform (CWT) of the second derivative of the smoothed ROI using the Mexican hat mother wavelet, then boundary-corrected and integrated into an RT/area/height peak table.

## When to use

Use when you have ROI chromatographic profiles (intensity vs. retention time) extracted from CE-MS or LC-MS data by the AriumMS ROI search, and need automatic peak detection with integration of apex RT, area, and height — especially for the narrow peaks typical of CE-MS separations.

## When NOT to use

- The full mzML/mzXML runs have not yet been reduced to regions of interest by the AriumMS ROI search — this skill operates on ROI traces, not raw files
- Input is already a detected-peak/feature table needing augmentation, false-positive filtering, or chemometric evaluation rather than peak detection
- You need MS/MS spectral networking to connect features across methods — explicitly listed as future development, not part of this workflow

## Inputs

- ROI chromatographic profiles (raw intensity-vs-retention-time traces) from the AriumMS ROI search on test CE-MS/LC-MS data
- MS data previously converted to .mzXML or .mzML via ProteoWizard msconvert

## Outputs

- detected_peaks.csv — one row per detected peak with non-null RT (apex), area, and height

## How to apply

Load each raw ROI intensity-vs-retention-time trace and apply Savitzky-Golay-style smoothing, estimating the local noise level from the difference between smoothed and original signal so peak criteria scale to the actual noise floor. Compute the second derivative of the smoothed ROI and run continuous wavelet transform peak detection with the Mexican hat mother wavelet, which suits the narrow peaks of CE-MS separations (AriumMSEval runs on MATLAB R2024a+, where the Wavelet Toolbox is a required dependency). Correct peak boundaries in two steps: first the friction-border rule, then a moving-standard-deviation criterion so boundaries stop where the signal rises above local noise. Finally integrate each detected peak to extract apex retention time, area, and height, and assemble all peaks into a single table written to detected_peaks.csv.

## Related tools

- **AriumMS** (Host app whose ROI search generates the ROI chromatographic profiles; the evaluation app (AriumMSEval) performs the CWT/Mexican-hat peak detection on those ROIs) — github.com/AdrianHaun/AriumMS
- **MATLAB Wavelet Toolbox** (Required toolbox providing the continuous wavelet transform with the Mexican hat mother wavelet used for peak detection in the AriumMS app version)
- **zmat** (Compression/decompression toolbox bundled as a dependency of the AriumMS MATLAB codebase) — http://github.com/fangq/zmat

## Evaluation signals

- detected_peaks.csv exists and is parseable
- every row of detected_peaks.csv carries non-null RT (apex), area, and height fields
- peak counts on the test ROIs are consistent with the expected detections for the AriumMS ROI search / feature detection and integration stage
- boundary refinement stops where the signal rises above the local noise floor per the moving-standard-deviation criterion
- noise-scaled criteria: peak detection thresholds are scaled to the noise estimated from the smoothed-minus-original residual, not absolute intensity cutoffs

## Limitations

- No version number stated for AriumMS, and no changelog found
- Requires MATLAB R2024a or newer plus Bioinformatics, Statistics and Machine Learning, Wavelet, Image Processing, Signal Processing, Parallel Computing, and Database toolboxes for the app version
- Method is oriented to the narrow peaks typical of CE-MS (nanoCEasy) separations; MS/MS spectral networking for connecting features across methods is not implemented and is future work

## Evidence

- [methods] CWT Mexican hat peak detection on second derivative: [evidence span withheld — anchor status: not_found]
- [methods] Two-step boundary correction: friction-border then moving-std: [evidence span withheld — anchor status: not_found]
- [discussion] Feature detection and integration within the AriumMS pipeline: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [discussion] Narrow CE-MS peaks motivate the wavelet choice: "All CE-MS methods show in general narrow peaks and benefit from the sensitivity of the nanoCEasy interface."
- [readme] Wavelet Toolbox required for the app version: "Required toolboxes for the app version: Bioinformatic Toolbox, Statistics And Machine Learning Toolbox, Wavelet Toolbox"
