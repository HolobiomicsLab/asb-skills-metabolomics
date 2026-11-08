---
name: gcxgc-4d-array-data-processing
description: Use when raw GC×GC–TOFMS CDF files must be imported as a 4D array and
  untargeted peak picking (ROI/blob detection) is needed before deconvolution, alignment,
  or annotation.
when_to_use_negative:
- Input is already a peak/feature table (e.g. from ChromaTOF or GC Image) rather than
  raw CDF data — GcDUO's Module I/II works from raw 4D reconstructions
- The goal is targeted quantification of known compounds with a library — use duoID/ConsProcessing
  (Modules III–V) instead of re-running ROI selection
- 1D GC–MS data without a second chromatographic dimension/modulation time — the 4D
  array and inverse-watershed blob detection assume GC×GC
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: R package providing readFolderCDF (4D array reconstruction) and procesData
    (rolling-window ROI segmentation by inverse watershed and peak picking with SNR/Gaussian
    filters)
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO is implemented and run
  repo: https://github.com/mariallr/GcDuo
- name: EBImage
  role: Required Bioconductor dependency installed via BiocManager before installing
    GcDUO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:8f95ae4fb5967fcfad45f6acc835b70fd440e3f044781649ba6764a091e2ae1f
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
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gcxgc-4d-array-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Processing of raw GC×GC–MS data (CDF/NetCDF folders) into a 4D intensity array (samples × m/z × RT1 × RT2) and extraction of blob features via GcDUO's readFolderCDF and procesData, which segments rolling windows, delineates ROIs with an inverse watershed algorithm, and retains only peaks passing signal-to-noise and Gaussian-shape filters.

## When to use

Use when raw GC×GC–TOFMS CDF files must be imported as a 4D array and untargeted peak picking (ROI/blob detection) is needed before deconvolution, alignment, or annotation.

## When NOT to use

- Input is already a peak/feature table (e.g. from ChromaTOF or GC Image) rather than raw CDF data — GcDUO's Module I/II works from raw 4D reconstructions
- The goal is targeted quantification of known compounds with a library — use duoID/ConsProcessing (Modules III–V) instead of re-running ROI selection
- 1D GC–MS data without a second chromatographic dimension/modulation time — the 4D array and inverse-watershed blob detection assume GC×GC

## Inputs

- Folder of raw GC×GC–MS CDF (NetCDF) files
- Modulation time (second-dimension time, seconds)
- m/z fragments range acquired (e.g. c(35, 440))
- User-set SNR thresholds for RT1 and RT2 and win_size (number of modulations per window)

## Outputs

- GcDuo S3 object: $data 4D intensity array (samples × m/z × retention 1 × retention 2), $time1d, $time2d, $mz, $files
- DuoResults object: $ind_peak per-sample detected feature table, $spec_ind per-sample feature spectra, $cor_spectra consensus spectra of common features
- Peak table exported as roi_peaks.csv

## How to apply

In R, install GcDUO from GitHub (devtools::install_github("mariallr/GcDUO"), requiring the EBImage Bioconductor package) and call readFolderCDF with the folder path, the modulation time in seconds (e.g. 2.5 s, the second-dimension time), and the acquired m/z range (e.g. c(35, 440)); this returns the GcDuo S3 object holding the 4D intensities array, time1d, time2d, mz, and file list. Visually verify the import with visualizeChrom3D (single sample contour plot) and visualizeChrom2D (TIC comparison across samples). Then run procesData with signalNoiseRatio1 and signalNoiseRatio2 thresholds (e.g. 50, settable per chromatographic dimension) and win_size in modulations (minimum recommended 3; increase if big peaks are present) so rolling windows are segmented and the inverse watershed algorithm delineates ROIs. Blobs are extracted as 3D peaks with 2D coordinates; only peaks with adequate signal-to-noise ratios in both dimensions and conforming to a Gaussian chromatographic form are retained, the rest being discarded as false positives. Collect the per-sample features (ind_peak) and spectra (spec_ind), plus cor_spectra consensus spectra for common features, from the returned DuoResults object.

## Related tools

- **GcDUO** (R package providing readFolderCDF (4D array reconstruction) and procesData (rolling-window ROI segmentation by inverse watershed and peak picking with SNR/Gaussian filters)) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO is implemented and run) — https://github.com/mariallr/GcDuo
- **EBImage** (Required Bioconductor dependency installed via BiocManager before installing GcDUO)

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440)); DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- roi_peaks.csv exists, is parseable, and contains detected peak identifiers with sample-level entries for each processed sample
- The GcDuo object exposes a 4D $data array ordered samples × m/z fragments × retention 1 × retention 2, with $time1d, $time2d, $mz and the CDF file list
- Retained blobs satisfy the configured SNR thresholds (e.g. 50 in both dimensions) and the Gaussian chromatographic form and minimum-point criteria; false positives with incomplete peak shape are discarded
- DuoResults contains $ind_peak (features sample by sample), $spec_ind (spectra sample by sample) and $cor_spectra (consensus spectra of common features)
- win_size is at least 3 modulations (the minimum recommended); increase it when big peaks are observed

## Limitations

- Decane was not detected at a signal-to-noise threshold of 3; it was only detected when the threshold was reduced to 1, so overly strict SNR settings miss small chromatographic signals
- Non-detected peaks correspond to small chromatographic signals with too few characteristic fragments to deconvolute
- R is not optimized for large memory loads or parallel processing on multiple cores, so large batches may need high-performance computing environments
- win_size below 3 modulations is not recommended; big peaks may require a larger window size

## Evidence
