---
name: mz-channel-variation-filtering
description: Use when deconvolving untargeted GC × GC–MS blobs (e.g., from CDF files
  loaded into a GcDUO object) where PARAFAC must be run on a reduced m/z dimension
  because full m/z ranges (e.g., 35–440) are dominated by uninformative channels.
when_to_use_negative:
- When the input is already a deconvoluted feature table or peak list — this filter
  operates on raw blob sub-tensors, not on extracted features
- When a targeted analysis already specifies which m/z fragments are informative (the
  top-5% variation rule is a blind, untargeted heuristic)
- When running PARAFAC2 across samples rather than per-blob PARAFAC — the filter is
  defined for per-sample, per-blob sub-tensor evaluation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: GcDUO
  role: R package whose Module III evaluates per-window m/z variation and selects
    the top 5% of m/z channels before blind PARAFAC deconvolution
  repo: https://github.com/mariallr/GcDuo
- name: PARAFAC
  role: decomposition applied to the variation-filtered sub-tensor to extract pure
    chemical signals from overlapping peaks
- name: multiway
  role: R package used for the PARAFAC implementation
- name: rrcov3way
  role: R package used for the robust multiway (PARAFAC) computations
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:22810e30a7562dcbd8c22138852ca2865da5b2415df109e68cd37fd124742e51
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# m/z channel variation filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A preprocessing filter inside GcDUO's Module III blind PARAFAC deconvolution: for each detected blob, variation within the m/z × RT2 × RT1 sub-tensor window is evaluated and only the top 5% of m/z channels are retained for component selection, reducing noise and computational cost.

## When to use

Use when deconvolving untargeted GC × GC–MS blobs (e.g., from CDF files loaded into a GcDUO object) where PARAFAC must be run on a reduced m/z dimension because full m/z ranges (e.g., 35–440) are dominated by uninformative channels.

## When NOT to use

- When the input is already a deconvoluted feature table or peak list — this filter operates on raw blob sub-tensors, not on extracted features
- When a targeted analysis already specifies which m/z fragments are informative (the top-5% variation rule is a blind, untargeted heuristic)
- When running PARAFAC2 across samples rather than per-blob PARAFAC — the filter is defined for per-sample, per-blob sub-tensor evaluation

## Inputs

- 4D GcDUO data array (samples x m/z fragments x RT1 x RT2) reconstructed from raw CDF files via readFolderCDF
- ROI/peak list produced by GcDUO's procesData (ind_peak table)
- Per-blob m/z × RT2 × RT1 sub-tensors

## Outputs

- Per-blob reduced tensor restricted to the top 5% most varying m/z channels, used for blind PARAFAC component selection
- Downstream deconvolution output (deconvolution_results.rds) with per-feature spectra, retention times, and peak assignments

## How to apply

After peak picking with procesData, extract the m/z × RT2 × RT1 sub-tensor for each sample and each detected blob, then evaluate the variation within each window and select only the top 5% of m/z channels (ranked by variation) before running blind PARAFAC. This reduces the data volume and focuses the decomposition on channels carrying the fragment signatures of the eluting compound, supporting the requirement that extracted components show a Gaussian chromatographic form with Tucker congruence above 0.9. Retain the selected channels only for component selection — the full sub-tensor still yields per-feature spectra, retention times, and peak assignments saved as deconvolution_results.rds.

## Related tools

- **GcDUO** (R package whose Module III evaluates per-window m/z variation and selects the top 5% of m/z channels before blind PARAFAC deconvolution) — https://github.com/mariallr/GcDuo
- **PARAFAC** (decomposition applied to the variation-filtered sub-tensor to extract pure chemical signals from overlapping peaks)
- **multiway** (R package used for the PARAFAC implementation)
- **rrcov3way** (R package used for the robust multiway (PARAFAC) computations)

## Evaluation signals

- All extracted PARAFAC components display a Gaussian chromatographic profile with Tucker congruence above 0.9
- Explained variance (R²) no longer improves as additional components are added
- Peak-area measurements correlate with gold-standard software (ChromaTOF) at R² = 0.9
- Duplicated features within the same sample are removed and cross-sample grouping at 60% dot-product spectral similarity yields coherent consensus spectra

## Limitations

- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted even after m/z filtering (e.g., decane was only detected when the signal-to-noise parameter was reduced from 3 to 1)
- The 5% cutoff is a fixed heuristic; peaks whose informative fragments fall below the variation threshold may be missed
- R is not optimized for large memory loads or parallel processing, so running PARAFAC per blob may necessitate high-performance computing environments

## Evidence
