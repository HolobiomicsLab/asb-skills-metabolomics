---
name: cross-sample-feature-grouping
description: Use when you have per-sample deconvolution results from untargeted GC×GC–MS
  data (e.g. GcDUO Module III output) and need to merge duplicate features into cross-sample
  consensus features with consensus spectra before annotation or integration.
when_to_use_negative:
- Only one sample is available — cross-sample grouping and consensus spectra cannot
  be formed; consensus spectra require common features between samples
- Input is already an aligned/annotated feature table — grouping precedes annotation
  (duoID) and integration (ConsProcessing), so re-grouping would duplicate work
- Features were produced by a different deconvolution pipeline whose spectra are not
  dot-product comparable with the per-sample PARAFAC-extracted spectra
edam_operation: http://edamontology.org/operation_3709
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: GcDUO
  role: R package that performs the duplicate-feature removal, cross-sample grouping
    via retention time and top m/z fragment matching at 60% dot-product similarity,
    and consensus spectrum computation within procesData
  repo: https://github.com/mariallr/GcDuo
- name: PARAFAC
  role: Upstream blind deconvolution that produces the per-sample pure-component spectra
    and retention times that are grouped across samples
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:a04ad4732d278c63f97bf36334451f73559bd0c2394d5c634af0dd0ca95527a3
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
  edam_operation: http://edamontology.org/operation_3709
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-sample-feature-grouping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Groups per-sample deconvolved GC×GC–MS features into consensus features by matching retention time and top m/z fragment with a dot-product spectral similarity cutoff of 60%, then computes the mean consensus spectrum per grouped feature. This bridges per-sample PARAFAC deconvolution and downstream annotation/quantification.

## When to use

Use when you have per-sample deconvolution results from untargeted GC×GC–MS data (e.g. GcDUO Module III output) and need to merge duplicate features into cross-sample consensus features with consensus spectra before annotation or integration.

## When NOT to use

- Only one sample is available — cross-sample grouping and consensus spectra cannot be formed; consensus spectra require common features between samples
- Input is already an aligned/annotated feature table — grouping precedes annotation (duoID) and integration (ConsProcessing), so re-grouping would duplicate work
- Features were produced by a different deconvolution pipeline whose spectra are not dot-product comparable with the per-sample PARAFAC-extracted spectra

## Inputs

- 4D GcDUO object ($data: samples × m/z fragments × retention 1 × retention 2) built from CDF files
- Per-sample deconvolution results (spectra, retention times, peak assignments) produced by blind PARAFAC in Module III
- Per-sample peak table and spectra from GcDUO procesData ($ind_peak, $spec_ind)

## Outputs

- Grouped cross-sample features with shared retention time and top m/z fragment
- Mean consensus spectrum for each grouped feature ($cor_spectra)
- Per-sample feature table retained with one-sample features flagged/preserved in Module III results

## How to apply

After running blind PARAFAC deconvolution on each sample's m/z × RT2 × RT1 sub-tensors, first remove duplicated features detected within the same sample. Then group remaining features across samples by retention time and top m/z fragment, requiring a dot-product spectral similarity of at least 60% for membership; compute the mean consensus spectrum for each grouped feature. In GcDUO, this is embedded in the procesData pipeline, whose output object stores $ind_peak (per-sample features), $spec_ind (per-sample spectra), and $cor_spectra (consensus spectra of common features between samples). Features detected in only one sample are not further evaluated in module V, but their data is preserved in the module III results, so the grouping step effectively gates which features proceed to annotation and quantification (ConsProcessing).

## Related tools

- **GcDUO** (R package that performs the duplicate-feature removal, cross-sample grouping via retention time and top m/z fragment matching at 60% dot-product similarity, and consensus spectrum computation within procesData) — https://github.com/mariallr/GcDuo
- **PARAFAC** (Upstream blind deconvolution that produces the per-sample pure-component spectra and retention times that are grouped across samples) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3); DuoResults$cor_spectra
```

## Evaluation signals

- Every consensus spectrum corresponds to features from ≥2 samples; features detected in only one sample are excluded from module V but preserved in module III results
- Grouped features share both retention time and the top m/z fragment, with intra-group dot-product spectral similarity ≥ 60%
- The procesData output object contains $cor_spectra entries for all common features between samples and $spec_ind entries for every per-sample feature
- Duplicated features within the same sample are removed before grouping, so no sample contributes more than one member per consensus group
- Downstream annotation (duoID) and quantification (ConsProcessing) consume only the grouped/consensus features, and validation against gold-standard software shows R² = 0.9 in peak area correlation

## Limitations

- The 60% dot-product similarity threshold is fixed in this workflow; low-abundance peaks with too few characteristic fragments to deconvolute may fail grouping and be missed (decane was only detected when the signal-to-noise parameter was reduced from 3 to 1)
- GcDUO is implemented in R, an environment not optimized for large memory loads or parallel processing across multiple cores, which may constrain batch grouping on large studies
- Features with cosine similarity ≤ 0.7 to the library are still grouped but not annotated — grouping does not guarantee compound identification

## Evidence

- [methods] Group features across samples by retention time and top m/z fragment using a dot-product spectral similarity threshold of 60%, and compute the mean consensus spectrum: [evidence span withheld — anchor status: not_found]
