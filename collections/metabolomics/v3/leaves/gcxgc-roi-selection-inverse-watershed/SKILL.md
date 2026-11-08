---
name: gcxgc-roi-selection-inverse-watershed
description: 'Use when processing raw GC×GC–MS (e.g., CDF files) to detect untargeted
  features: you need to segment chromatographic matrices into ROIs and filter false-positive
  peaks by S/N and Gaussian shape before deconvolution.'
when_to_use_negative:
- Input is already a feature/peak table from commercial software (e.g. ChromaTOF peak
  tables) rather than raw CDF files — ROI selection operates on the reconstructed
  4D intensity array.
- You need targeted quantification of known compounds; GcDUO ROI selection is designed
  for untargeted feature extraction where ROIs are unknown.
- Peaks are expected to be smaller than the S/N threshold allows — e.g. decane was
  only detected when the signal-to-noise parameter was reduced from 3 to 1, so overly
  strict S/N settings will drop real but weak signals.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: Provides readFolderCDF to build the 4D GcDuo object and procesData to segment
    rolling windows, delineate ROIs with the inverse watershed algorithm, and filter
    blobs by signal-to-noise ratio and Gaussian form
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Implementation platform; install via devtools::install_github("mariallr/GcDUO")
    with the EBImage dependency from Bioconductor
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:a2309e2ea1bd22e5032b8f757a09d7084d0fe93a9b30d972e94eb161e2347216
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/GcDuo
  repo_url: https://github.com/mariallr/GcDuo
  tools:
  - GcDUO
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gcxgc-roi-selection-inverse-watershed

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Segments GC×GC–MS data into rolling windows of modulations and delineates regions of interest (ROIs) with the inverse watershed algorithm, extracting candidate blobs (3D peaks with 2D coordinates), then retains only blobs with adequate signal-to-noise ratios in both chromatographic dimensions and a Gaussian chromatographic form.

## When to use

Use when processing raw GC×GC–MS (e.g., CDF files) to detect untargeted features: you need to segment chromatographic matrices into ROIs and filter false-positive peaks by S/N and Gaussian shape before deconvolution.

## When NOT to use

- Input is already a feature/peak table from commercial software (e.g. ChromaTOF peak tables) rather than raw CDF files — ROI selection operates on the reconstructed 4D intensity array.
- You need targeted quantification of known compounds; GcDUO ROI selection is designed for untargeted feature extraction where ROIs are unknown.
- Peaks are expected to be smaller than the S/N threshold allows — e.g. decane was only detected when the signal-to-noise parameter was reduced from 3 to 1, so overly strict S/N settings will drop real but weak signals.

## Inputs

- Raw GC×GC–MS data as CDF/NetCDF files in a folder
- Modulation time (second-dimension run time, seconds)
- m/z fragment range acquired (e.g. c(35, 440))
- Signal-to-noise thresholds for dimension 1 (signalNoiseRatio1) and dimension 2 (signalNoiseRatio2)
- Window size in modulations (win_size, minimum recommended 3)

## Outputs

- DuoResults object with $ind_peak: table of detected features sample by sample
- DuoResults object with $spec_ind: spectra of detected features sample by sample
- DuoResults object with $cor_spectra: consensus spectra of common features between samples
- Peak table (e.g. roi_peaks.csv) with detected peak identifiers and sample-level entries

## How to apply

Load raw NetCDF/CDF files with GcDuo::readFolderCDF (specifying modulationTime for the second dimension, e.g. 2.5 s, and the acquired m/z range such as c(35, 440)) to obtain a GcDuo S3 object holding a 4D intensities array (samples x m/z x RT1 x RT2), time1d, time2d, mz, and the file list. Run GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3): the algorithm segments the dataset into rolling windows of modulations, identifies the prominent points of the matrix with the inverse watershed algorithm within these windows to delineate ROIs, and segments them into individual peaks (3D peaks with 2D coordinates). Set win_size to the number of modulations per window — increase it if large peaks are present, but keep a minimum of 3 modulations. Retain only blobs whose peaks pass the user-set signal-to-noise ratios in both the first and second chromatographic dimensions and that conform to a Gaussian form with sufficient points; blobs failing these criteria are discarded as false positives. Collect ind_peak (per-sample detected features) and spec_ind (per-sample spectra) from the returned DuoResults object and export the peak table (e.g. roi_peaks.csv).

## Related tools

- **GcDUO** (Provides readFolderCDF to build the 4D GcDuo object and procesData to segment rolling windows, delineate ROIs with the inverse watershed algorithm, and filter blobs by signal-to-noise ratio and Gaussian form) — https://github.com/mariallr/GcDuo
- **R** (Implementation platform; install via devtools::install_github("mariallr/GcDUO") with the EBImage dependency from Bioconductor) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- roi_peaks.csv exists, is parseable, and contains detected peak identifiers with sample-level entries for each processed sample.
- Each retained blob passes both signalNoiseRatio1 and signalNoiseRatio2 thresholds (e.g. 50) as configured.
- Retained peaks conform to a Gaussian chromatographic form with the minimum-point criteria; incomplete peak shapes or sub-threshold S/N blobs are discarded as false positives.
- $ind_peak and $spec_ind in the DuoResults object are populated for every sample in the $files list of the GcDuo object.
- win_size was kept at or above 3 modulations per window (increased only for large peaks).

## Limitations

- Very small chromatographic signals with too few characteristic fragments may not survive: decane was not detected at S/N threshold 3 and was only detected when the threshold was reduced to 1.
- R is not optimized for handling large memory loads or parallel processing in multiple cores, so batch processing may necessitate high-performance computing environments.
- The window size (win_size) must be tuned; if big peaks are observed it should be increased, but below 3 modulations per window the algorithm is not recommended.

## Evidence
