---
name: gcxgc-roi-subtensor-extraction
description: Use when processing raw GC × GC–MS CDF files with GcDUO and you need
  per-peak sub-tensors (from a 4D samples × m/z × RT1 × RT2 array) to feed blind PARAFAC
  deconvolution.
when_to_use_negative:
- Input is already a peak table or feature matrix (e.g. from ChromaTOF) — this skill
  extracts ROIs from raw 4D chromatographic data only
- Data are 1D GC–MS rather than comprehensive 2D GC × GC–MS with a defined modulation
  time
- You need targeted analysis with prior knowledge of components — blind PARAFAC ROI
  extraction is designed for untargeted use without previous knowledge of the samples
edam_topics: []
tools:
- name: GcDUO
  role: R package providing readFolderCDF (builds the 4D array from CDF files), visualizeChrom2D/3D
    for visual checks, and procesData (peak picking) whose $ind_peak list defines
    the ROIs to extract
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:b8aeeea46c11648c367ef2373d3fe6300f12fb45d77c64cb79892f5770bd2223
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
  - multiway
  - rrcov3way
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# gcxgc-roi-subtensor-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts region-of-interest (ROI) m/z × RT2 × RT1 sub-tensors for each detected peak blob from a GcDUO 4D data array in GC × GC–MS, as the input preparation step before blind PARAFAC deconvolution.

## When to use

Use when processing raw GC × GC–MS CDF files with GcDUO and you need per-peak sub-tensors (from a 4D samples × m/z × RT1 × RT2 array) to feed blind PARAFAC deconvolution.

## When NOT to use

- Input is already a peak table or feature matrix (e.g. from ChromaTOF) — this skill extracts ROIs from raw 4D chromatographic data only
- Data are 1D GC–MS rather than comprehensive 2D GC × GC–MS with a defined modulation time
- You need targeted analysis with prior knowledge of components — blind PARAFAC ROI extraction is designed for untargeted use without previous knowledge of the samples

## Inputs

- Folder of CDF (NetCDF) raw GC × GC–MS files
- Modulation time (seconds) and m/z acquisition range (e.g. 35–440)
- Signal-to-noise thresholds for RT1 and RT2 (e.g. 50) and window size in modulations (e.g. 3)

## Outputs

- GcDUO S3 object with $data (4D array samples × m/z × RT1 × RT2), $time1d, $time2d, $mz, $files
- $ind_peak: table of detected features per sample (ROI/peak list)
- $spec_ind: spectra of features detected sample by sample
- Per-blob m/z × RT2 × RT1 sub-tensors restricted to the top 5% most-varying m/z channels

## How to apply

First load the raw data with GcDUO::readFolderCDF, specifying the modulation time (second-dimension time, e.g. 2.5 s) and the acquired m/z range (e.g. c(35, 440)) to obtain the 4D array ordered samples × m/z × retention 1 × retention 2. Run peak picking with GcDUO::procesData, setting signalNoiseRatio1 and signalNoiseRatio2 (e.g. 50) and win_size (minimum 3 modulations) to obtain the $ind_peak table of features detected sample by sample. For each sample and each detected blob, extract the m/z × RT2 × RT1 sub-tensor around the peak and evaluate the variation within the window, retaining only the top 5% of m/z channels for component selection. This ROI delineation relies on rolling windows plus the inverse watershed algorithm, which segments the prominent points of the matrix into individual peaks; only peaks with adequate signal-to-noise and Gaussian form are retained downstream.

## Related tools

- **GcDUO** (R package providing readFolderCDF (builds the 4D array from CDF files), visualizeChrom2D/3D for visual checks, and procesData (peak picking) whose $ind_peak list defines the ROIs to extract) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- Retained peaks exhibit adequate signal-to-noise ratios and conform to a Gaussian form
- Each extracted sub-tensor covers the blob's m/z × RT2 × RT1 window with only the top 5% of m/z channels selected by within-window variation
- $ind_peak lists features sample by sample and win_size is at least the recommended 3 modulations (increase for big peaks)
- Downstream blind PARAFAC on each sub-tensor yields components with Gaussian profiles and Tucker congruence above 0.9, with R² no longer improving when components are added

## Limitations

- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted (e.g. decane was only detected when the signal-to-noise parameter was reduced from 3 to 1)
- win_size tuning matters: increase for big peaks, but the minimum recommended value is 3 modulations
- R is not optimized for large memory loads or parallel processing, so handling full 4D arrays may necessitate high-performance computing environments

## Evidence
