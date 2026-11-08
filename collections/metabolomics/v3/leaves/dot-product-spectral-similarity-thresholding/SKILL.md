---
name: dot-product-spectral-similarity-thresholding
description: Use when consolidating per-sample deconvoluted GC×GC–MS feature lists
  into cross-sample consensus features, and you need a spectral-similarity rule to
  merge duplicate/variant detections of the same compound across samples.
when_to_use_negative:
- When spectra come from a different acquisition platform or ionization mode where
  the 60% dot-product threshold has not been validated — this cutoff is specific to
  GcDUO's GC×GC–TOFMS setting
- When the goal is library-based compound identification — use cosine similarity (threshold
  0.7) against an MSP library instead of cross-sample dot-product grouping
- When input is already an aligned cross-sample feature table; this step is part of
  deconvolution output consolidation, not re-grouping of aligned tables
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: R package whose Module III implements the within-sample duplicate removal,
    cross-sample feature grouping by retention time and top m/z fragments with 60%
    dot-product spectral similarity, and consensus-spectrum computation
  repo: https://github.com/mariallr/GcDuo
- name: PARAFAC
  role: Produces the per-sample deconvoluted spectra that are the input to the cross-sample
    grouping step
- name: R
  role: Language in which GcDUO (and hence this grouping step) is implemented; results
    saved as deconvolution_results.rds
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:adeefd7c423ab4c9483ab138d3fb5b6b43c831b4b3692a8124bf33565a4cbb5d
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
  - R
  - multiway
  - rrcov3way
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# dot-product-spectral-similarity-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A cross-sample feature-grouping step in GcDUO's Module III deconvolution: after blind per-sample PARAFAC deconvolution, features from different samples are grouped by retention time and their top m/z fragments using a dot-product spectral similarity threshold of 60%, and a mean consensus spectrum is computed per group.

## When to use

Use when consolidating per-sample deconvoluted GC×GC–MS feature lists into cross-sample consensus features, and you need a spectral-similarity rule to merge duplicate/variant detections of the same compound across samples.

## When NOT to use

- When spectra come from a different acquisition platform or ionization mode where the 60% dot-product threshold has not been validated — this cutoff is specific to GcDUO's GC×GC–TOFMS setting
- When the goal is library-based compound identification — use cosine similarity (threshold 0.7) against an MSP library instead of cross-sample dot-product grouping
- When input is already an aligned cross-sample feature table; this step is part of deconvolution output consolidation, not re-grouping of aligned tables

## Inputs

- Per-sample deconvolution results from blind PARAFAC (per-feature spectra, retention times, peak assignments)
- Per-sample m/z fragment spectra from deconvoluted blobs (top 5% most variable m/z channels)
- 4D GcDUO data array reconstructed from raw CDF files (samples x m/z x RT1 x RT2)

## Outputs

- Grouped cross-sample features with mean consensus spectra (cor_spectra)
- deconvolution_results.rds containing per-feature spectra, retention times, and peak assignments
- Feature table where single-sample-only features are flagged for exclusion from module V evaluation

## How to apply

After running blind PARAFAC per sample/blob (components accepted only when Gaussian with Tucker congruence > 0.9 and R² stops improving), first remove duplicated features detected within the same sample. Then group features across samples by matching retention time and the top m/z fragments, computing dot-product spectral similarity between each pair of spectra; group only pairs meeting the 60% similarity threshold. For each accepted group, compute the mean consensus spectrum, which becomes the feature's representative spectrum for downstream annotation (e.g., duoID cosine matching against an MSP library) and quantification (ConsProcessing). Features detected in only one sample are not further evaluated in module V, though their data is preserved in module III results.

## Related tools

- **GcDUO** (R package whose Module III implements the within-sample duplicate removal, cross-sample feature grouping by retention time and top m/z fragments with 60% dot-product spectral similarity, and consensus-spectrum computation) — https://github.com/mariallr/GcDuo
- **PARAFAC** (Produces the per-sample deconvoluted spectra that are the input to the cross-sample grouping step)
- **R** (Language in which GcDUO (and hence this grouping step) is implemented; results saved as deconvolution_results.rds)

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- Grouped features show a mean consensus spectrum computed from member features with dot-product similarity ≥ 60% against the group's top m/z fragments
- No duplicated features remain within the same sample after the deduplication step preceding grouping
- Consensus spectra feed module IV annotation and achieve ≥ 89% correct peak identification across the two validation datasets, and R² = 0.9 peak-area correlation vs. ChromaTOF
- Features detected in only one sample are correctly preserved in module III output but excluded from module V evaluation
- Output is stored as deconvolution_results.rds with per-feature spectra, retention times, and peak assignments

## Limitations

- The 60% dot-product threshold is fixed in GcDUO's workflow; no guidance is given for tuning it to other datasets
- Features detected in only one sample are excluded from module V evaluation, so low-abundance single-sample compounds drop out of cross-sample analysis
- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted and therefore cannot be reliably grouped
- R is not optimized for large memory loads or multi-core parallel processing, which may limit scaling of the grouping step on large batches

## Evidence

- [results] cross-sample grouping threshold: [evidence span withheld — anchor status: not_found]
- [results] deconvolution output artifact: [evidence span withheld — anchor status: not_found]
