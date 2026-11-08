---
name: chromatographic-data-integration-qc
description: Use when you need batch-consistent peak areas/intensities per feature
  across GC×GC–MS samples (e.g. a serial dilution QC series) and must verify quantitative
  agreement with gold-standard software like ChromaTOF.
when_to_use_negative:
- Input is already a curated peak/feature table from another tool — GcDUO's ConsProcessing
  expects the raw CDF-derived GcDuoObject and its own DuoResults.
- You only need peak picking or visualization without cross-sample integration and
  quantification (use visualizeChrom2D/3D or procesData alone).
- Data is 1D GC–MS rather than comprehensive two-dimensional GC×GC–MS with a defined
  modulation time.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: R package whose ConsProcessing module integrates picked/annotated peaks into
    per-feature area and intensity matrices across the batch
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO is implemented and where correlation (Pearson r) and
    dilution-curve R^2 statistics are computed
  repo: https://github.com/mariallr/GcDuo
- name: ChromaTOF
  role: Gold-standard commercial software whose peak areas serve as the reference
    for Pearson-correlation validation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:274c16fabd171d4597c0c577ad6f003eb3c1466a71c3bf25dd5ba7eb832ddcaf
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
  - dplyr
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# chromatographic-data-integration-qc

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates peak picking and annotation results into a per-feature peak area matrix across a GC×GC–TOFMS batch using GcDUO's ConsProcessing, then validates quantitative consistency (Pearson correlation vs. ChromaTOF, dilution-series linearity).

## When to use

Use when you need batch-consistent peak areas/intensities per feature across GC×GC–MS samples (e.g. a serial dilution QC series) and must verify quantitative agreement with gold-standard software like ChromaTOF.

## When NOT to use

- Input is already a curated peak/feature table from another tool — GcDUO's ConsProcessing expects the raw CDF-derived GcDuoObject and its own DuoResults.
- You only need peak picking or visualization without cross-sample integration and quantification (use visualizeChrom2D/3D or procesData alone).
- Data is 1D GC–MS rather than comprehensive two-dimensional GC×GC–MS with a defined modulation time.

## Inputs

- GcDuoObject from GcDUO::readFolderCDF (4D array from CDF files, with modulation time e.g. 2.5 s and mzRange e.g. c(35, 440))
- DuoResults object from GcDUO::procesData (ind_peak, spec_ind, cor_spectra)
- Library object built from an MSP file via getMSP/spectramatrix (comp, exactmass, formula, inchikey, ri + spectra matrix)
- ChromaTOF gold-standard peak area table for correlation-based QC

## Outputs

- finalGcDuo object with $peaks (peak summary), $area (area per peak per sample), $intensity, and $spectra
- Pearson correlation statistic between GcDUO and ChromaTOF peak areas (e.g. 0.904 for the Breath Mix 1 ppm sample)
- Mean dilution-curve R^2 across peaks (e.g. 0.950) and per-peak area table (breath_mix_linearity_results.csv)

## How to apply

After peak picking (GcDUO::procesData with signal-to-noise thresholds in RT1/RT2 and win_size >= 3 modulations) and annotation (duoID, cosine match_cutoff = 0.7), run GcDUO::ConsProcessing with the imported MSP library and a win_width (e.g. 20) to obtain the $area and $intensity matrices, which integrate each common feature per sample. Features detected in only one sample are excluded from further evaluation in module V (their data remains in module III results). For QC, extract the target sample's peak-area column, correlate against gold-standard (ChromaTOF) areas via Pearson correlation, and for dilution series fit log-transformed area vs. dilution level per peak, averaging R^2 across peaks. In the Breath Mix validation this reproduced Pearson r = 0.904 at 1 ppm and mean dilution-curve R^2 > 0.950.

## Related tools

- **GcDUO** (R package whose ConsProcessing module integrates picked/annotated peaks into per-feature area and intensity matrices across the batch) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO is implemented and where correlation (Pearson r) and dilution-curve R^2 statistics are computed) — https://github.com/mariallr/GcDuo
- **ChromaTOF** (Gold-standard commercial software whose peak areas serve as the reference for Pearson-correlation validation)

## Examples

```
finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20)
```

## Evaluation signals

- Pearson correlation between GcDUO and ChromaTOF peak areas for the reference sample reproduces ~0.904 (Breath Mix 1 ppm).
- Mean dilution-curve R^2 (log area vs. dilution level, averaged over peaks) exceeds 0.950.
- $area and $intensity matrices in finalGcDuo have one row per feature and one column per CDF sample, with no missing integration for features common across samples.
- Features detected in only one sample are absent from module V evaluation but retained in module III ($ind_peak) results.
- All integrated peaks previously passed the Gaussian-form and signal-to-noise filters of procesData.

## Limitations

- Non-detected peaks correspond to small chromatographic signals with too few characteristic fragments to deconvolute.
- GcDUO is implemented in R, an environment not optimized for large memory loads or multi-core parallel processing; big batches may need high-performance computing environments.
- Annotation/integration quality depends on the in-house library: compounds without cosine similarity > 0.7 return no annotation, and compounds missing from the library cannot be integrated as identified features.

## Evidence

- [readme] finalGcDuo object with $area and $intensity matrices: "We obtain the finalGcDuo object with: - `$peaks`: summary of the peaks detected - `$area`: area for each peak and sample - `$intensity`: intensity for each peak and sample"
- [readme] ConsProcessing invocation parameters: "finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20)"
- [abstract] Validation against gold standard: "GcDUO was validated against the gold-standard software for comprehensive GC, demonstrating a high correlation (R2 = 0.9) in peak area measurements"
- [discussion] Single-sample features excluded in module V: "features detected in only one sample are not further evaluated in module V, but their data is preserved in the results of module III"
- [results] Small signals fail deconvolution: "The non-detected peaks correspond to small chromatographic signals with too few characteristic fragments to deconvolute."
