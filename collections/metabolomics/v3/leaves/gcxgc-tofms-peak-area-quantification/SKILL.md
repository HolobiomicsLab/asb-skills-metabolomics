---
name: gcxgc-tofms-peak-area-quantification
description: Use when comparing an open-source GC×GC–MS peak-processing pipeline against
  a commercial gold standard on serial-dilution QC data, or when validating that extracted
  peak areas are linearly proportional to analyte concentration.
when_to_use_negative:
- Input is already a peak-area feature table — this skill covers extraction from raw
  CDF files through GcDUO processing, not re-analysis of exported tables
- Data are single-dimension GC–MS or LC–MS rather than comprehensive two-dimensional
  GC×GC–TOFMS with a defined modulation time
- The goal is annotation/deconvolution evaluation (e.g. cosine-similarity library
  matching) rather than quantitative area concordance and dilution linearity
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: R package used to read CDF files (readFolderCDF), pick peaks (procesData),
    and produce the consensus peak-area matrix (ConsProcessing) across the dilution
    series
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Implementation platform for GcDUO and for computing Pearson correlation, log-transformed
    linear regressions, and mean R² statistics
  repo: https://github.com/mariallr/GcDuo
- name: ChromaTOF
  role: Gold-standard commercial software whose peak areas provide the reference values
    for the Pearson correlation (r = 0.904)
- name: dplyr
  role: R package for manipulating the per-peak area table before computing correlation
    and dilution-curve statistics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:e201261788a259e585d7873135024ff73ca1a91c2d740d0abc9a4c367f41b99b
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gcxgc-tofms-peak-area-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce quantitative peak-area agreement between GcDUO and gold-standard ChromaTOF on GC×GC–TOFMS serial-dilution data: correlate 1 ppm peak areas (Pearson r ≈ 0.904) and fit log-area vs. dilution-level linear regressions per peak to obtain a mean R² ≈ 0.950 dilution-curve linearity.

## When to use

Use when comparing an open-source GC×GC–MS peak-processing pipeline against a commercial gold standard on serial-dilution QC data, or when validating that extracted peak areas are linearly proportional to analyte concentration.

## When NOT to use

- Input is already a peak-area feature table — this skill covers extraction from raw CDF files through GcDUO processing, not re-analysis of exported tables
- Data are single-dimension GC–MS or LC–MS rather than comprehensive two-dimensional GC×GC–TOFMS with a defined modulation time
- The goal is annotation/deconvolution evaluation (e.g. cosine-similarity library matching) rather than quantitative area concordance and dilution linearity

## Inputs

- Folder of CDF files from a GC×GC–TOFMS serial dilution QC dataset (in-house Breath Mix, 1 ppm plus dilution levels)
- Modulation time (second-dimension time in seconds) and acquired m/z range
- MSP spectral library converted via getMSP/spectramatrix into a GcDUO lib object (compound names, exactmass, formula, inchikey, ri, spectra matrix)
- Gold-standard peak-area table from ChromaTOF for the reference sample

## Outputs

- finalGcDuo object with $area (peak area per feature and sample), $intensity, $spectra, $peaks
- Pearson's correlation coefficient of 1 ppm GcDUO vs. ChromaTOF peak areas (target ~0.904)
- Mean per-peak dilution-curve R² from linear regression of log-transformed area vs. dilution level (target ~0.950)
- breath_mix_linearity_results.csv with per-peak area table and the two statistics

## How to apply

Import the raw GC×GC–TOFMS CDF serial-dilution folder with GcDUO::readFolderCDF, specifying the modulation time (second-dimension time, e.g. 2.5 s) and the acquired m/z range (e.g. 35–440). Run GcDUO::procesData for peak picking across the dilution series, setting signalNoiseRatio1 and signalNoiseRatio2 thresholds and win_size = 3 modulations (minimum recommended). Run GcDUO::ConsProcessing with the imported MSP library to obtain the $area matrix of peak areas per feature per sample. Extract the 1 ppm sample column and compute Pearson's correlation of GcDUO areas vs. ChromaTOF areas (expected r ≈ 0.904); then, per peak, regress log-transformed area on dilution level, compute R² for each dilution curve, and average (expected mean R² ≈ 0.950). Write the per-peak area table and the two statistics to breath_mix_linearity_results.csv.

## Related tools

- **GcDUO** (R package used to read CDF files (readFolderCDF), pick peaks (procesData), and produce the consensus peak-area matrix (ConsProcessing) across the dilution series) — https://github.com/mariallr/GcDuo
- **R** (Implementation platform for GcDUO and for computing Pearson correlation, log-transformed linear regressions, and mean R² statistics) — https://github.com/mariallr/GcDuo
- **ChromaTOF** (Gold-standard commercial software whose peak areas provide the reference values for the Pearson correlation (r = 0.904))
- **dplyr** (R package for manipulating the per-peak area table before computing correlation and dilution-curve statistics)

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440)); DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3); finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20)
```

## Evaluation signals

- Pearson's correlation of GcDUO vs. ChromaTOF peak areas for the 1 ppm sample reproduces ~0.904 (consistent with the abstract's reported high correlation, R² = 0.9)
- Mean dilution-curve R² across peaks reproduces ~0.950 after linear regression of log-transformed area vs. dilution level
- finalGcDuo$area is a features × samples matrix covering every dilution level, with per-peak entries matching finalGcDuo$peaks ids
- Per-peak R² values concentrate near 1 for well-detected compounds; peaks failing Gaussian-form or signal-to-noise filters (e.g. low-intensity signals with too few characteristic fragments) are absent or non-linear
- breath_mix_linearity_results.csv contains both the per-peak area table and the two headline statistics

## Limitations

- Signals with too few characteristic fragments to deconvolute (small chromatographic peaks) may not be detected, biasing area comparisons toward larger peaks
- win_size must be increased for big peaks; the minimum recommended value is 3 modulations, and the signal-to-noise thresholds directly control which features are retained (decane was only detected when SNR was reduced from 3 to 1)
- GcDUO in R is not optimized for large memory loads or multi-core parallel processing; large batches may need high-performance computing environments
- No GcDUO/R version numbers are stated, so exact reproducibility may vary across package versions

## Evidence

- [results] For the Breath Mix dilution series, GcDUO peak areas at 1 ppm showed a Pearson's correlation of 0.904 with ChromaTOF, and the dilutions showed strong linearity with a mean r2 above 0.950.: [evidence span withheld — anchor status: not_found]
