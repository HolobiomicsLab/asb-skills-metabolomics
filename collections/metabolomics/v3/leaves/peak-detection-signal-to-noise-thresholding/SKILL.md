---
name: peak-detection-signal-to-noise-thresholding
description: Use when raw GC×GC–MS CDF files must be converted into detected peak
  features, and you need to control detection sensitivity via first- and second-dimension
  signal-to-noise thresholds (e.g. benchmarking detection rates like the 27 fruitybeer
  peaks).
when_to_use_negative:
- Input is already a peak table or feature-intensity matrix rather than raw CDF files
  — peak picking with S/N thresholds is redundant and cannot be applied.
- The goal is deconvolution of co-eluting compounds (PARAFAC/PARAFAC2) or spectral
  annotation (duoID) rather than initial ROI/peak detection.
- Data are 1D GC–MS or LC–MS rather than comprehensive two-dimensional GC×GC–MS with
  a defined modulation time.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: R package providing readFolderCDF (CDF import into a 4D array) and procesData
    (S/N-thresholded peak picking over rolling windows with inverse watershed ROI
    delineation)
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO is implemented and executed; installed via devtools::install_github
    with EBImage as a required dependency
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:811764f62832480bbdfd4083ee31abd8884a924ffb4b5fdc232fc2c31807c819
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# peak-detection-signal-to-noise-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Signal-to-noise thresholding in GcDUO for detecting region-of-interest peaks in GC×GC–MS data: the procesData function applies dimension-specific S/N cutoffs (RT1 and RT2) within rolling windows, retaining only peaks with adequate signal-to-noise and Gaussian form. It recovers chromatographic features from raw 4D CDF data prior to deconvolution and annotation.

## When to use

Use when raw GC×GC–MS CDF files must be converted into detected peak features, and you need to control detection sensitivity via first- and second-dimension signal-to-noise thresholds (e.g. benchmarking detection rates like the 27 fruitybeer peaks).

## When NOT to use

- Input is already a peak table or feature-intensity matrix rather than raw CDF files — peak picking with S/N thresholds is redundant and cannot be applied.
- The goal is deconvolution of co-eluting compounds (PARAFAC/PARAFAC2) or spectral annotation (duoID) rather than initial ROI/peak detection.
- Data are 1D GC–MS or LC–MS rather than comprehensive two-dimensional GC×GC–MS with a defined modulation time.

## Inputs

- Folder of raw GC×GC–MS CDF (NetCDF) files
- Modulation time in seconds (second-dimension run time, e.g. 2.5 s)
- Acquired m/z fragment range (e.g. c(35, 440))
- Signal-to-noise thresholds for RT1 and RT2 (signalNoiseRatio1, signalNoiseRatio2)
- Window size in modulations (win_size, minimum recommended 3)

## Outputs

- DuoResults$ind_peak: table of features detected sample by sample
- DuoResults$spec_ind: spectra of the features detected sample by sample
- DuoResults$cor_spectra: consensus spectra of common features between samples

## How to apply

Import the raw CDF files with GcDuo::readFolderCDF specifying the modulation time (e.g. 2.5 s) and the acquired m/z range (e.g. 35–440), producing the 4D GcDuo object. Run GcDuo::procesData with signalNoiseRatio1 and signalNoiseRatio2 controlling the intensity thresholds in the first and second chromatographic dimensions (50/50 was used for the fruitybeer benchmark) and win_size = 3, the number of modulations per rolling window (minimum recommended is 3; increase for big peaks). Within each window only the top 5% of m/z channels by variation are selected, prominent points are identified with the inverse watershed algorithm to delineate ROIs, and only peaks with adequate signal-to-noise ratios and Gaussian form are retained. The output DuoResults contains $ind_peak (features detected sample by sample), $spec_ind (sample-wise spectra), and $cor_spectra (consensus spectra of common features). If expected low-abundance compounds (like decane in the article) are missed, lower the S/N thresholds — decane was only detected when the threshold was reduced from 3 to 1 — accepting more noise peaks in exchange.

## Related tools

- **GcDUO** (R package providing readFolderCDF (CDF import into a 4D array) and procesData (S/N-thresholded peak picking over rolling windows with inverse watershed ROI delineation)) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO is implemented and executed; installed via devtools::install_github with EBImage as a required dependency) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- DuoResults$ind_peak is a non-empty table listing features per sample; $spec_ind and $cor_spectra are populated for detected features.
- On the fruitybeer benchmark (S/N 50/50, win_size 3), 85% of the 27 peaks reported by the study authors are detected, with three peaks detected but later left unannotated (cosine similarity ≤ 0.7).
- Every retained peak exhibits adequate signal-to-noise ratio and conforms to a Gaussian form; non-detected peaks should correspond to small chromatographic signals with too few characteristic fragments to deconvolute.
- Sensitivity check: lowering the S/N parameter recovers known low-abundance compounds — decane was only detected when the signal-to-noise parameter was reduced from 3 to 1.
- win_size must be at least 3 modulations; larger peaks may require increasing it.

## Limitations

- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted and go undetected even at benchmark thresholds.
- Detection is threshold-sensitive: known compounds like decane are missed at higher S/N settings and only recovered when the threshold is lowered.
- R is not optimized for large memory loads or multi-core parallel processing, so large GC×GC–MS datasets may necessitate high-performance computing environments.

## Evidence
