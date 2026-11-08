---
name: gcxgc-roi-peak-picking-signal-to-noise-thresholding
description: Use when raw GC×GC–TOFMS CDF files have been imported into a GcDuoObject
  4D array and detected features must be extracted sample-by-sample before deconvolution,
  annotation, or peak-area comparison.
when_to_use_negative:
- Input is already a peak table (e.g. exported from ChromaTOF) rather than raw CDF
  data — peak picking is redundant; go straight to alignment/regression
- 'Data are 1D GC–MS rather than comprehensive 2D GC×GC: the modulation-time and win_size
  parameters and 4D ROI segmentation do not apply'
- Goal is final per-peak areas/intensities for quantification — use ConsProcessing
  downstream on DuoResults instead of re-running peak picking
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: GcDUO
  role: Provides readFolderCDF (4D array construction), procesData (ROI segmentation,
    peak picking with SNR thresholds), and visualizeChrom3D/visualizeChrom2D for visual
    verification
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Implementation platform for the GcDUO package; installed via devtools::install_github
    with the EBImage dependency from BiocManager
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:a3f24976c97ddd649c0f778aa4d2f9bfbb7d42329b8b672929a3bf20e8457c98
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gcxgc-roi-peak-picking-signal-to-noise-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

ROI-based peak picking for GC×GC–MS 4D data arrays in GcDUO: segment chromatograms into rolling windows, delineate ROIs with the inverse watershed algorithm, and retain peaks passing dimension-specific signal-to-noise thresholds and Gaussian-shape criteria.

## When to use

Use when raw GC×GC–TOFMS CDF files have been imported into a GcDuoObject 4D array and detected features must be extracted sample-by-sample before deconvolution, annotation, or peak-area comparison.

## When NOT to use

- Input is already a peak table (e.g. exported from ChromaTOF) rather than raw CDF data — peak picking is redundant; go straight to alignment/regression
- Data are 1D GC–MS rather than comprehensive 2D GC×GC: the modulation-time and win_size parameters and 4D ROI segmentation do not apply
- Goal is final per-peak areas/intensities for quantification — use ConsProcessing downstream on DuoResults instead of re-running peak picking

## Inputs

- GcDuoObject 4D array (samples x m/z fragments x RT1 x RT2) built by readFolderCDF from CDF files, with modulationTime and mzRange specified
- Signal-to-noise thresholds for RT1 (signalNoiseRatio1) and RT2 (signalNoiseRatio2)
- Window size in modulations (win_size)

## Outputs

- DuoResults object with $ind_peak (feature table per sample), $spec_ind (per-sample spectra) and $cor_spectra (consensus spectra of features common across samples)

## How to apply

Starting from the GcDuoObject produced by readFolderCDF (modulationTime = 2.5 s, mzRange = c(35, 440)), run GcDuo::procesData with signalNoiseRatio1 and signalNoiseRatio2 set per dimension (e.g. 50/50) and win_size = 3, the minimum recommended number of modulations per window (increase it if big peaks are observed). The algorithm segments the dataset into rolling windows, evaluates m/z variation, selects only the top 5% most varying m/z channels, and uses the inverse watershed algorithm to delineate ROIs and segment individual peaks; only peaks with adequate signal-to-noise ratios and a Gaussian form are retained. The returned DuoResults contains $ind_peak, $spec_ind and $cor_spectra; thresholds must be tuned empirically because overly strict SNR settings miss small signals (decane was only detected when SNR was reduced from 3 to 1), while overly loose settings admit noise. Verify peak picking visually with visualizeChrom3D/visualizeChrom2D before proceeding to duoID annotation (cosine similarity cutoff 0.7) and ConsProcessing quantification.

## Related tools

- **GcDUO** (Provides readFolderCDF (4D array construction), procesData (ROI segmentation, peak picking with SNR thresholds), and visualizeChrom3D/visualizeChrom2D for visual verification) — https://github.com/mariallr/GcDuo
- **R** (Implementation platform for the GcDUO package; installed via devtools::install_github with the EBImage dependency from BiocManager) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- DuoResults object contains non-empty $ind_peak, $spec_ind and $cor_spectra tables
- All retained peaks conform to Gaussian form and exceed the configured per-dimension SNR thresholds
- win_size is at least 3 modulations (increase only when large peaks are observed)
- Known low-abundance standards (e.g. decane) are recovered when thresholds are lowered, indicating sensitivity rather than over-filtering
- Peak count is plausible when overlaid on visualizeChrom3D/visualizeChrom2D contour plots of the same samples

## Limitations

- win_size has a recommended minimum of 3 modulations; smaller windows risk missing peaks
- Very small chromatographic signals with too few characteristic fragments cannot be deconvoluted even if detected
- Threshold sensitivity: decane was not detected at SNR threshold 3 and only appeared when reduced to 1
- GcDUO runs in R, which is not optimized for large memory loads or multi-core parallelism; large GC×GC datasets may require high-performance computing

## Evidence
