---
name: rolling-window-modulation-segmentation
description: Use when starting from raw GC×GC–MS CDF files (4D intensities array with
  two retention-time dimensions and m/z fragments) and you need untargeted ROI selection
  and blob extraction before peak deconvolution, with user-set S/N thresholds and
  win_size in modulations.
when_to_use_negative:
- Input is already a peak table or feature matrix from commercial software (e.g. ChromaTOF
  peak tables) rather than raw CFD/CDF data — no ROI segmentation is needed
- Data are 1D GC–MS without a second chromatographic dimension or modulation time
  — the rolling-window two-dimensional segmentation does not apply
- The goal is library-based annotation or PARAFAC/PARAFAC2 deconvolution of already-segmented
  ROIs — use downstream GcDUO modules (duoID, ConsProcessing) instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: GcDUO
  role: R package whose Module II (procesData) performs rolling-window segmentation,
    inverse-watershed ROI delineation, blob extraction, and S/N + Gaussian-form filtering
  repo: https://github.com/mariallr/GcDuo
- name: EBImage
  role: Required Bioconductor dependency of GcDUO (image-processing support for the
    watershed-based segmentation)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:c9a95f66a65be32e7a97e6e0d562e5d3d1845f6c98f14e8cd1fde7c5d237712d
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# rolling-window-modulation-segmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Segment GC×GC–MS chromatographic data into rolling windows of modulations and delineate regions of interest (ROIs) with the inverse watershed algorithm, extracting candidate blobs (3D peaks with 2D coordinates) filtered by signal-to-noise ratio and Gaussian peak form. This is Module II of GcDUO's raw-data processing pipeline, preceding PARAFAC/PARAFAC2 deconvolution.

## When to use

Use when starting from raw GC×GC–MS CDF files (4D intensities array with two retention-time dimensions and m/z fragments) and you need untargeted ROI selection and blob extraction before peak deconvolution, with user-set S/N thresholds and win_size in modulations.

## When NOT to use

- Input is already a peak table or feature matrix from commercial software (e.g. ChromaTOF peak tables) rather than raw CFD/CDF data — no ROI segmentation is needed
- Data are 1D GC–MS without a second chromatographic dimension or modulation time — the rolling-window two-dimensional segmentation does not apply
- The goal is library-based annotation or PARAFAC/PARAFAC2 deconvolution of already-segmented ROIs — use downstream GcDUO modules (duoID, ConsProcessing) instead

## Inputs

- Folder of raw GC×GC–MS CDF files (NetCDF format)
- Modulation time in seconds (second-column time), e.g. 2.5
- Acquired m/z fragment range, e.g. c(35, 440)
- GcDuoObject S3 object from readFolderCDF containing $data (4D intensities array), $time1d, $time2d, $mz, $files

## Outputs

- DuoResults object with $ind_peak (per-sample detected features with 2D peak coordinates), $spec_ind (per-sample feature spectra), $cor_spectra (consensus spectra of common features)
- roi_peaks.csv peak table with detected peak identifiers and sample-level entries

## How to apply

Load raw data with GcDuo::readFolderCDF, specifying the modulation time (e.g. 2.5 s, the second-dimension time) and the acquired m/z range (e.g. c(35, 440)) to obtain the GcDuoObject holding the 4D intensities array (samples x m/z x RT1 x RT2), time1d, time2d, mz, and the CDF file list. Call GcDuo::procesData with signalNoiseRatio1 and signalNoiseRatio2 (thresholds for intensity in RT1 and RT2 respectively; the paper's example uses 50) and win_size, the number of modulations per rolling window — keep at least 3 modulations and increase win_size when large peaks are observed. Within each window, the inverse watershed algorithm identifies the prominent points of the matrix to precisely delineate ROIs and segment them into individual blobs; window variation is additionally used to select only the top 5% of m/z channels during component selection. Retain only blobs whose peaks exhibit adequate signal-to-noise in both chromatographic dimensions and conform to a Gaussian form; discard incomplete or sub-threshold peaks as false positives. The returned DuoResults object provides $ind_peak (features detected sample by sample), $spec_ind (per-sample spectra), and $cor_spectra (consensus spectra of features common between samples).

## Related tools

- **GcDUO** (R package whose Module II (procesData) performs rolling-window segmentation, inverse-watershed ROI delineation, blob extraction, and S/N + Gaussian-form filtering) — https://github.com/mariallr/GcDuo
- **EBImage** (Required Bioconductor dependency of GcDUO (image-processing support for the watershed-based segmentation))

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- roi_peaks.csv exists, is parseable, and contains detected peak identifiers with sample-level entries for every processed sample
- DuoResults$ind_peak contains per-sample feature entries with 2D chromatographic coordinates; DuoResults$spec_ind and $cor_spectra are populated correspondingly
- Retention of blobs is consistent with the configured signalNoiseRatio1 = signalNoiseRatio2 = 50 thresholds and the Gaussian chromatographic-form criterion; incomplete or sub-threshold peaks are absent
- win_size respects the minimum recommendation of 3 modulations, and window size was increased if large peaks were present
- Windows contribute only the top 5% most-variable m/z channels for component selection

## Limitations

- Small chromatographic signals may be missed: decane was only detected when the signal-to-noise parameter was reduced from 3 to 1, indicating S/N threshold sensitivity
- win_size below the recommended minimum of 3 modulations is discouraged; big peaks may require increasing win_size
- GcDUO is implemented in R, which is not optimized for large memory loads or multi-core parallel processing, so large GC×GC–MS batches may necessitate high-performance computing environments

## Evidence
