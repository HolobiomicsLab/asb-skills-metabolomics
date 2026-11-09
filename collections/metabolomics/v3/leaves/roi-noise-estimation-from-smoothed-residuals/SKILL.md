---
name: roi-noise-estimation-from-smoothed-residuals
description: Use when raw CE-MS or LC-MS ROI chromatographic profiles (intensity-vs-retention-time
  traces) must have a per-ROI noise level estimated from smoothed-minus-original residuals
  before peak criteria (e.g., CWT peak picking) are applied.
when_to_use_negative:
- The data are already a processed feature/peak table (e.g., post feature detection
  and integration) — noise estimation applies to raw chromatographic ROI profiles
  before peak calling
- A single-platform tool (XCMS Online, MetaboAnalyst, MZmine) is already configured
  with its own noise/centWave thresholds and ROI augmentation is not the goal
- The traces are not chromatographic profiles (e.g., MS/MS spectra or spectroscopic
  data), where smoothed-minus-residual noise scaling is not the described criterion
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: AriumMS
  role: MATLAB toolbox whose ROI preprocessing estimates the local noise from the
    smoothed-vs-original residual to scale ROI feature detection and integration criteria
  repo: github.com/AdrianHaun/AriumMS
- name: msconvert (ProteoWizard)
  role: Converts vendor MS data to .mzXML/.mzML, the input format AriumMS expects
    before ROI search and noise-scaled peak detection
  repo: http://proteowizard.sourceforge.net/download.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00216-023-04715-6
    title: Augmented region of interest for untargeted metabolomics mass spectrometry
      (AriumMS) of multi-platform-based CE-MS and LC-MS data
  source_content_hash: sha256:a9cfa0e9d777c9e5760eb47e707a84853ed48c47c7940ca2804f37e334f9e4d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AriumMS
  - msconvert (ProteoWizard)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
  techniques:
  - CE-MS
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# roi-noise-estimation-from-smoothed-residuals

## Summary

Estimates the local noise floor of each chromatographic region of interest (ROI) by taking the difference between a smoothed signal and the original raw trace, so that downstream peak-detection criteria in the AriumMS/AriumMSEval pipeline are scaled to the actual noise of CE-MS/LC-MS intensity-vs-retention-time profiles.

## When to use

Use when raw CE-MS or LC-MS ROI chromatographic profiles (intensity-vs-retention-time traces) must have a per-ROI noise level estimated from smoothed-minus-original residuals before peak criteria (e.g., CWT peak picking) are applied.

## When NOT to use

- The data are already a processed feature/peak table (e.g., post feature detection and integration) — noise estimation applies to raw chromatographic ROI profiles before peak calling
- A single-platform tool (XCMS Online, MetaboAnalyst, MZmine) is already configured with its own noise/centWave thresholds and ROI augmentation is not the goal
- The traces are not chromatographic profiles (e.g., MS/MS spectra or spectroscopic data), where smoothed-minus-residual noise scaling is not the described criterion

## Inputs

- Raw ROI chromatographic profiles (intensity-vs-retention-time traces) from CE-MS and LC-MS runs
- MS data in .mzXML or .mzML format (converted with msconvert)
- Smoothing parameters (e.g., Savitzky-Golay-style window/order) for the ROI preprocessing

## Outputs

- Per-ROI local noise-level estimates used to scale peak-detection criteria
- Noise-scaled peak criteria consumed by the CWT (Mexican hat) peak picker and the moving-std boundary refinement
- Downstream detected-peak table (detected_peaks.csv) with RT, area, and height per peak

## How to apply

First convert vendor MS data to .mzXML or .mzML with msconvert (ProteoWizard), then extract the ROI chromatographic profiles. Apply Savitzky-Golay-style smoothing to each ROI and compute the difference between the smoothed and the original signal; this residual approximates the local noise because smoothing suppresses real signal while retaining stochastic measurement error. Use this noise estimate to scale peak-detection thresholds so that candidate peak centers — found later via the second derivative and continuous wavelet transform with the Mexican hat mother wavelet — are judged against the actual noise floor rather than a fixed cutoff. The same noise floor feeds the boundary-refinement step, where a moving standard deviation criterion stops peak boundaries where the signal rises above local noise. In AriumMS this runs inside MATLAB (R2024a or newer) using the Wavelet, Signal Processing, and Statistics toolboxes on ROI regions found with the ROIpeaks/MSroiaug functions.

## Related tools

- **AriumMS** (MATLAB toolbox whose ROI preprocessing estimates the local noise from the smoothed-vs-original residual to scale ROI feature detection and integration criteria) — github.com/AdrianHaun/AriumMS
- **msconvert (ProteoWizard)** (Converts vendor MS data to .mzXML/.mzML, the input format AriumMS expects before ROI search and noise-scaled peak detection) — http://proteowizard.sourceforge.net/download.html

## Evaluation signals

- detected_peaks.csv exists, is parseable, and every row carries non-null RT, area, and height fields
- Peak counts on the test ROIs are consistent with the expected detections for the AriumMS ROI search / feature detection and integration stage
- Noise estimates track the magnitude of the smoothed-minus-original residuals per ROI (noisy ROIs get proportionally higher thresholds)
- Peak boundaries stop where the signal rises above the local noise (moving standard deviation criterion), and the friction-border rule is applied before the moving-std refinement

## Limitations

- CE-MS peaks are narrow (especially with the nanoCEasy interface), so smoothing windows and the Mexican hat wavelet width must be chosen to suit narrow peaks; overly wide smoothing will inflate residuals and misestimate noise
- AriumMS requires MATLAB R2024a or newer plus the Wavelet, Signal Processing, Statistics, and other listed toolboxes
- No version number or changelog is stated for AriumMS, so reproducibility of the noise-estimation parameters across releases is not documented

## Evidence

- [full_text] noise estimation from smoothed residuals: [evidence span withheld — anchor status: not_found]
- [full_text] noise-scaled boundary refinement: [evidence span withheld — anchor status: not_found]
- [discussion] narrow CE-MS peaks motivation: "All CE-MS methods show in general narrow peaks and benefit from the sensitivity of the nanoCEasy interface."
- [discussion] AriumMS pipeline placement: "AriumMS contains ROI search, preprocessing, feature detection and integration, false-positive filter, scaling, and transformation followed by the augmentation and various chemometric data evaluation"
- [readme] input format conversion requirement: "For MS Data conversion to .mzXML or .mzML file format use msconvert, distributed with the ProteoWizard Project"
