---
name: chromatographic-peak-snr-filtering
description: Use when filtering candidate peaks detected from GC×GC–MS 4D intensity
  arrays (CDF-derived GcDuo objects) by signal-to-noise and peak-shape criteria before
  deconvolution, to discard false positives such as incomplete or sub-threshold signals.
when_to_use_negative:
- Input is already a curated peak/feature table rather than raw GC×GC–MS CDF-derived
  4D intensity data — SNR and Gaussian-form filtering of blobs assumes Module II ROI
  detection is being (re)run.
- 'Analytes of interest are known to be very low abundance: strict SNR thresholds
  can miss them entirely (decane required lowering the SNR from 3 to 1), so a targeted
  or relaxed-threshold approach is preferable.'
- Peaks are strongly non-Gaussian by nature (e.g., saturated or badly resolved chromatographic
  signals) — the Gaussian-form requisite will discard them even if signal is strong.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: R package whose Module II implements rolling-window segmentation, inverse-watershed
    ROI selection, and SNR/Gaussian-form blob filtering via procesData
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO is implemented and run
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:aec4ef07f48082e7e309b07e326185e4229c4a5b3bd74e0bc21a35b7520ea700
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# chromatographic-peak-snr-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A peak-filtering step from GcDUO's Module II in which candidate blobs (3D peaks with 2D coordinates) detected by the inverse-watershed ROI selection on GC×GC–MS data are retained only if they exhibit adequate signal-to-noise ratios in both chromatographic dimensions and conform to a Gaussian chromatographic form.

## When to use

Use when filtering candidate peaks detected from GC×GC–MS 4D intensity arrays (CDF-derived GcDuo objects) by signal-to-noise and peak-shape criteria before deconvolution, to discard false positives such as incomplete or sub-threshold signals.

## When NOT to use

- Input is already a curated peak/feature table rather than raw GC×GC–MS CDF-derived 4D intensity data — SNR and Gaussian-form filtering of blobs assumes Module II ROI detection is being (re)run.
- Analytes of interest are known to be very low abundance: strict SNR thresholds can miss them entirely (decane required lowering the SNR from 3 to 1), so a targeted or relaxed-threshold approach is preferable.
- Peaks are strongly non-Gaussian by nature (e.g., saturated or badly resolved chromatographic signals) — the Gaussian-form requisite will discard them even if signal is strong.

## Inputs

- GcDuo S3 object from readFolderCDF containing the 4D intensities array (samples x m/z fragments x retention time 1 x retention time 2), time1d, time2d, mz, and CDF file list
- User-set SNR thresholds for RT1 and RT2 (signalNoiseRatio1, signalNoiseRatio2)
- Rolling window size in modulations (win_size, minimum recommended 3)

## Outputs

- DuoResults object with $ind_peak (per-sample detected feature table) and $spec_ind (per-sample feature spectra)
- Filtered set of blobs (3D peaks with 2D coordinates) retained as true features; false positives discarded
- Downstream peak table (e.g., roi_peaks.csv) with detected peak identifiers and sample-level entries

## How to apply

After readFolderCDF produces the GcDuo object (4D intensities array, time1d, time2d, mz, file list), run GcDuo::procesData with user-set signalNoiseRatio1 and signalNoiseRatio2 thresholds (e.g., 50 in the documented example) applied to the first and second chromatographic dimensions respectively, and win_size = 3 modulations per rolling window (increase win_size for big peaks; 3 is the recommended minimum). The inverse-watershed algorithm segments ROIs within these windows and extracts blobs (3D peaks with 2D coordinates); only blobs whose peaks pass both SNR thresholds and satisfy the Gaussian-form and minimum-point criteria are retained, while the rest are discarded as false positives. Retained features are returned sample-by-sample as ind_peak and spec_ind in the DuoResults object. The threshold is dataset-dependent: in the validation work, decane was only detected when the signal-to-noise parameter was reduced from 3 to 1, so low-abundance analytes may require threshold relaxation at the cost of more noise peaks.

## Related tools

- **GcDUO** (R package whose Module II implements rolling-window segmentation, inverse-watershed ROI selection, and SNR/Gaussian-form blob filtering via procesData) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO is implemented and run) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- roi_peaks.csv exists, is parseable, and contains detected peak identifiers with sample-level entries for each processed sample.
- Retained blobs all meet the configured signalNoiseRatio1/signalNoiseRatio2 thresholds (e.g., 50) in both chromatographic dimensions and the Gaussian-form/minimum-point criteria.
- DuoResults object contains non-empty $ind_peak and $spec_ind tables with per-sample entries.
- Sensitivity check: known low-abundance standards (e.g., decane) are detected when the SNR threshold is appropriately relaxed, confirming thresholds are not over-stringent.
- Discarded candidates correspond to sub-threshold or non-Gaussian (incomplete peak shape) signals rather than well-formed Gaussian peaks.

## Limitations

- SNR thresholds are dataset- and analyte-dependent: decane was only detected when the signal-to-noise parameter was reduced from 3 to 1, so overly strict settings miss small chromatographic signals.
- Small chromatographic signals with too few characteristic fragments to deconvolute are not detected/recovered.
- win_size below 3 modulations is not recommended; big peaks require increasing win_size, otherwise segmentation/filtering may fail.
- R is not optimized for large memory loads or multi-core parallel processing; large GC×GC–MS batches may need high-performance computing environments.

## Evidence
