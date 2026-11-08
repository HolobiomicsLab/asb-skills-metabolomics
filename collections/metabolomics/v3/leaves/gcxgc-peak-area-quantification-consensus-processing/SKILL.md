---
name: gcxgc-peak-area-quantification-consensus-processing
description: Use when you have GC×GC–MS CDF raw files processed through GcDUO peak
  picking/annotation and need per-feature peak areas and intensities per sample for
  quantification or software benchmarking.
when_to_use_negative:
- Input is already a feature/peak-area table with no raw CDF files — ConsProcessing
  requires the raw GcDuoObject 4D array
- Peaks have not been deconvoluted and annotated yet — run procesData and duoID first
- You need 1D GC–MS quantification — GcDUO targets comprehensive two-dimensional GC×GC–MS
  data with a known modulation time
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: Provides the ConsProcessing function that computes per-peak areas and intensities
    across the batch from the deconvoluted results
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Implementation platform; installed via devtools::install_github with EBImage
    dependency from BiocManager
- name: ChromaTOF
  role: Commercial gold-standard software whose peak-area tables serve as the benchmark
    reference for R² correlation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:07b93edf0ef6515d5ab882781635b3e1f597d9572f73f788af8b544c465b1507
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
  - ChromaTOF
  - GcDUO
  - R
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# gcxgc-peak-area-quantification-consensus-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs GcDUO's consensus processing module (ConsProcessing) to integrate deconvoluted GC×GC–MS features into per-peak areas and intensities across a sample batch, enabling peak-area quantification benchmarked against gold-standard software like ChromaTOF.

## When to use

Use when you have GC×GC–MS CDF raw files processed through GcDUO peak picking/annotation and need per-feature peak areas and intensities per sample for quantification or software benchmarking.

## When NOT to use

- Input is already a feature/peak-area table with no raw CDF files — ConsProcessing requires the raw GcDuoObject 4D array
- Peaks have not been deconvoluted and annotated yet — run procesData and duoID first
- You need 1D GC–MS quantification — GcDUO targets comprehensive two-dimensional GC×GC–MS data with a known modulation time

## Inputs

- GcDuoObject (4D array from readFolderCDF on CDF files: samples × m/z × RT1 × RT2)
- DuoResults object from procesData and duoID (detected peaks, consensus spectra, annotations)
- MSP spectral library imported via getMSP/spectramatrix
- Optional: gold-standard peak-area table (e.g. ChromaTOF) for benchmark comparison

## Outputs

- finalGcDuo$peaks: summary of peaks detected
- finalGcDuo$area: area for each peak and sample
- finalGcDuo$intensity: intensity for each peak and sample
- finalGcDuo$spectra: spectra for each peak and sample
- Regression summary (R², slope, intercept, n peaks) when benchmarked against ChromaTOF

## How to apply

After running procesData (peak picking, e.g. signalNoiseRatio1/2 = 50, win_size = 3) and duoID (annotation, match_cutoff = 0.7), apply GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20) to align consensus peaks across samples and extract areas. Use win_width as the average peak width parameter. The returned finalGcDuo object supplies $peaks, $area, $intensity, and $spectra tables; features detected in only one sample are excluded from module V evaluation but preserved in module III results. To benchmark quantification, load a ChromaTOF peak-area table for the same samples, align peaks, and compute linear-regression R² of GcDUO vs ChromaTOF areas, accepting reproduction at R² ≥ 0.88 (reported ~0.9).

## Related tools

- **GcDUO** (Provides the ConsProcessing function that computes per-peak areas and intensities across the batch from the deconvoluted results) — https://github.com/mariallr/GcDuo
- **R** (Implementation platform; installed via devtools::install_github with EBImage dependency from BiocManager)
- **ChromaTOF** (Commercial gold-standard software whose peak-area tables serve as the benchmark reference for R² correlation)

## Examples

```
finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20)
```

## Evaluation signals

- finalGcDuo contains non-empty $area and $intensity tables with one row per consensus peak and one column per sample
- Benchmark regression of GcDUO vs ChromaTOF peak areas yields R² ≥ 0.88 (reported 0.9/0.904)
- Features detected in only one sample are absent from the consensus evaluation but present in module III results
- win_width parameter (e.g. 20) is set to a plausible average peak width; plotInspect/PlotPeak EIC for a known peakid recovers the expected chromatographic shape

## Limitations

- Features detected in only one sample are not further evaluated in module V; small chromatographic signals with too few characteristic fragments may not be deconvoluted at all
- GcDUO is implemented in R, which is not optimized for large memory loads or parallel processing; large datasets may need high-performance computing environments
- Detection sensitivity depends on signal-to-noise thresholds — e.g. decane was only detected when the signal-to-noise parameter was reduced from 3 to 1

## Evidence
