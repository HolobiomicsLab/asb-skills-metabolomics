---
name: parafac-tensor-deconvolution-gcxgc-ms
description: Use when raw GC × GC–MS data (e.g., CDF files folded into a 4D samples
  × m/z × RT1 × RT2 array) contain overlapping peaks that must be deconvoluted into
  pure chemical signals without prior knowledge of sample composition.
when_to_use_negative:
- The input is already a deconvoluted peak/feature table rather than raw multi-way
  chromatographic data
- The analysis goal is targeted quantification with known reference spectra — annotation
  via duoID (cosine similarity > 0.7 against an msp library) is more appropriate
- Data are 1D GC–MS or non-chromatographic; the tri-linear PARAFAC tensor structure
  (m/z × RT2 × RT1) is required
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: Open-source R software whose Module III performs the blind per-sample PARAFAC
    deconvolution, duplicate filtering, and cross-sample consensus grouping of GC
    × GC–MS features
  repo: https://github.com/mariallr/GcDuo
- name: PARAFAC
  role: Tri-linear tensor decomposition algorithm applied iteratively to each peak
    sub-tensor to separate overlapping chemical signals
  repo: https://github.com/mariallr/GcDuo
- name: multiway
  role: R package implementing PARAFAC used by GcDUO for the tensor decomposition
- name: rrcov3way
  role: R package supporting robust multi-way analysis used in GcDUO's deconvolution
    stack
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:d6b42dd95e4b2c9f49ee7fdd8b059834a4f8f4158c816bbb320bee0f3dd8e227
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
  - PARAFAC
  - multiway
  - rrcov3way
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 24
---

# parafac-tensor-deconvolution-gcxgc-ms

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Blind PARAFAC deconvolution of GC × GC–MS peaks as implemented in GcDUO's Module III: iteratively fitting PARAFAC on m/z × RT2 × RT1 sub-tensors, increasing the component count until explained variance (R²) stops improving and all components are Gaussian with Tucker congruence > 0.9, followed by duplicate-feature removal and cross-sample consensus grouping.

## When to use

Use when raw GC × GC–MS data (e.g., CDF files folded into a 4D samples × m/z × RT1 × RT2 array) contain overlapping peaks that must be deconvoluted into pure chemical signals without prior knowledge of sample composition.

## When NOT to use

- The input is already a deconvoluted peak/feature table rather than raw multi-way chromatographic data
- The analysis goal is targeted quantification with known reference spectra — annotation via duoID (cosine similarity > 0.7 against an msp library) is more appropriate
- Data are 1D GC–MS or non-chromatographic; the tri-linear PARAFAC tensor structure (m/z × RT2 × RT1) is required

## Inputs

- 4D GC × GC–MS data array (samples × m/z fragments × RT1 × RT2) reconstructed from raw CDF files by GcDUO Module I
- ROI/peak list produced by peak picking with GcDUO's procesData function (ind_peak, spec_ind tables)
- Signal-to-noise thresholds and window size used during peak picking (e.g., signalNoiseRatio1 = 50, win_size = 3)

## Outputs

- deconvolution_results.rds containing per-feature spectra, retention times, and peak assignments
- Per-component PARAFAC loading vectors (chromatographic profiles and mass spectra) with R² and Tucker congruence values
- Consensus spectra of grouped features across samples (cor_spectra)

## How to apply

Load the 4D data array with GcDUO's readFolderCDF (with modulation time and m/z range such as c(35, 440)) and the ROI/peak list from procesData. For each sample and each detected blob, extract the m/z × RT2 × RT1 sub-tensor; evaluate variation within the window and retain only the top 5% of m/z channels for component selection. Run blind PARAFAC starting with a low number of components, increasing iteratively until R² ceases to improve and every extracted component shows a Gaussian chromatographic profile with Tucker congruence above 0.9. Then remove duplicated features within the same sample, group features across samples by retention time and top m/z fragment using a dot-product spectral similarity threshold of 60%, and compute the mean consensus spectrum per grouped feature. Save per-feature spectra, retention times, and peak assignments as deconvolution_results.rds.

## Related tools

- **GcDUO** (Open-source R software whose Module III performs the blind per-sample PARAFAC deconvolution, duplicate filtering, and cross-sample consensus grouping of GC × GC–MS features) — https://github.com/mariallr/GcDuo
- **PARAFAC** (Tri-linear tensor decomposition algorithm applied iteratively to each peak sub-tensor to separate overlapping chemical signals) — https://github.com/mariallr/GcDuo
- **multiway** (R package implementing PARAFAC used by GcDUO for the tensor decomposition)
- **rrcov3way** (R package supporting robust multi-way analysis used in GcDUO's deconvolution stack)

## Examples

```
GcDuoObject <- GcDuo::readFolderCDF(folderPath = path, modulationTime = 2.5, mzRange = c(35, 440)); DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- All extracted PARAFAC components display a Gaussian chromatographic profile with Tucker congruence above 0.9
- The model's explained variance (R²) no longer improves when additional components are added, indicating the correct component count
- Peak-area measurements correlate with the gold-standard software ChromaTOF at R² = 0.9
- No duplicated features remain within the same sample after filtering; cross-sample grouping is stable at dot-product spectral similarity ≥ 60%
- Small chromatographic signals with too few characteristic fragments may fail to deconvolute (e.g., decane was only detected when signal-to-noise was reduced from 3 to 1), so expected low-abundance features may be legitimately absent

## Limitations

- Non-detected peaks correspond to small chromatographic signals with too few characteristic fragments to deconvolute
- R's limited memory handling and parallelization may necessitate high-performance computing environments for large batches; no specific hardware or version specifications are provided
- PARAFAC is computationally costly and typically requires knowledge of the number of components; GcDUO addresses this with iterative component search, which adds runtime
- Features detected in only one sample are not further evaluated in module V, though their data is preserved in the module III results

## Evidence

- [results] iterative component-count criterion: [evidence span withheld — anchor status: not_found]
