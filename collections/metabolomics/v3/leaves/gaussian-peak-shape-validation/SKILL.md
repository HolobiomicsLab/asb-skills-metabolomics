---
name: gaussian-peak-shape-validation
description: Use when performing untargeted GC × GC–MS peak picking from raw CDF files
  and false-positive blobs (incomplete peak shapes, sub-threshold S/N) must be discarded
  before deconvolution by PARAFAC/PARAFAC2.
when_to_use_negative:
- Input is already a feature/peak table from another tool (e.g. ChromaTOF, GC Image
  output) rather than raw CDF files — this filter is part of GcDUO's blob detection
  on raw 4D intensities
- Analytes of interest are small chromatographic signals with too few characteristic
  fragments to deconvolute — Gaussian/SNR filtering may discard them (non-detected
  peaks in the fruitybeer dataset were such small signals)
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: Implements ROI selection via inverse watershed and enforces the signal-to-noise
    and Gaussian peak-shape filters on detected blobs
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO's peak filtering and blob validation are executed
- name: EBImage
  role: Required dependency for GcDUO's image-based inverse-watershed ROI segmentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:9c1f3546dd6153feea394d9877864ace15ad3821a62c89a3748c72ff8b30a04b
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
  - EBImage
  - GcDUO
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gaussian-peak-shape-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validation step in GcDUO's Module II ROI selection for GC × GC–MS data: after inverse-watershed blob detection, candidate 3D peaks (with 2D coordinates) are retained only if their signal-to-noise ratios (SNR1/SNR2 thresholds, e.g. 50 in both dimensions) pass and their chromatographic form conforms to a Gaussian shape with a minimum number of modulation points.

## When to use

Use when performing untargeted GC × GC–MS peak picking from raw CDF files and false-positive blobs (incomplete peak shapes, sub-threshold S/N) must be discarded before deconvolution by PARAFAC/PARAFAC2.

## When NOT to use

- Input is already a feature/peak table from another tool (e.g. ChromaTOF, GC Image output) rather than raw CDF files — this filter is part of GcDUO's blob detection on raw 4D intensities
- Analytes of interest are small chromatographic signals with too few characteristic fragments to deconvolute — Gaussian/SNR filtering may discard them (non-detected peaks in the fruitybeer dataset were such small signals)

## Inputs

- GcDuo S3 object from readFolderCDF ($data 4D array samples × m/z × RT1 × RT2, $time1d, $time2d, $mz, $files)
- User-set parameters: signalNoiseRatio1, signalNoiseRatio2 (e.g. 50), win_size (≥3 modulations)

## Outputs

- DuoResults object with $ind_peak (per-sample detected features passing S/N and Gaussian-form filters) and $spec_ind (per-sample spectra)
- $cor_spectra: consensus spectra of common features between samples

## How to apply

After loading raw CDF files into a GcDuo object via readFolderCDF (4D intensities array, time1d, time2d, mz, file list), run GcDuo::procesData with user-set signalNoiseRatio1 and signalNoiseRatio2 thresholds (e.g. 50 in both chromatographic dimensions) and win_size = 3 modulations per rolling window; the inverse watershed algorithm delineates ROIs and segments them into individual blobs. Retain only blobs whose peaks exceed the SNR thresholds in RT1 and RT2 and whose extracted signal conforms to a Gaussian chromatographic form; this Gaussian requirement is also imposed later on all components extracted by the blind PARAFAC. Blobs failing shape or S/N criteria (e.g. peaks spanning fewer than the recommended minimum of 3 modulations) are discarded as false positives. Parameter tuning matters: for example, decane was only detected when the signal-to-noise parameter was reduced from 3 to 1, so thresholds should be adjusted to the intensity regime of expected analytes.

## Related tools

- **GcDUO** (Implements ROI selection via inverse watershed and enforces the signal-to-noise and Gaussian peak-shape filters on detected blobs) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO's peak filtering and blob validation are executed)
- **EBImage** (Required dependency for GcDUO's image-based inverse-watershed ROI segmentation)

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- roi_peaks.csv exists, is parseable, and contains detected peak identifiers with sample-level entries for every processed sample
- Every retained blob in $ind_peak satisfies the configured signalNoiseRatio1/2 thresholds (e.g. 50 in both RT1 and RT2)
- Retained peak traces conform to a Gaussian chromatographic form and span at least the recommended minimum of 3 modulations (win_size = 3)
- Blobs with incomplete peak shapes or sub-threshold S/N are absent from the retained set (false positives discarded)
- Validation against gold-standard software (ChromaTOF) shows high correlation (R2 = 0.9) in peak area measurements; features detected in only one sample are preserved in module III results but excluded from module V

## Limitations

- Very low-abundance compounds may be missed at strict S/N thresholds — decane was only detected when the signal-to-noise parameter was reduced from 3 to 1
- win_size must be tuned to peak width: increase it for big peaks, but the minimum recommended value is 3 modulations
- Small chromatographic signals with too few characteristic fragments to deconvolute are lost as non-detected peaks
- R is not optimized for large memory loads or multi-core parallel processing, so high-performance computing may be needed for large batches

## Evidence
