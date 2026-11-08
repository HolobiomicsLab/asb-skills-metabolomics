---
name: tucker-congruence-component-validation
description: Use when performing blind (no prior component knowledge) PARAFAC deconvolution
  of m/z × RT2 × RT1 sub-tensors from untargeted GC × GC–TOFMS data and you must decide
  the correct number of components and reject non-chemical (non-Gaussian) loadings.
when_to_use_negative:
- When component identity/number is already known (supervised PARAFAC); the blind
  iterative fitting is designed for samples with no prior knowledge
- When the analysis goal is peak alignment across samples rather than per-blob deconvolution
  — PARAFAC2 relaxes the tri-linearity constraint for that purpose
- When input is already a peak/feature table from another tool (e.g. ChromaTOF) rather
  than raw CFD chromatographic data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3855
tools:
- name: GcDUO
  role: R package whose Module III wraps the blind per-sample PARAFAC deconvolution
    with the Tucker congruence and Gaussian validation criteria
  repo: https://github.com/mariallr/GcDuo
- name: PARAFAC
  role: Chemometric tri-linear decomposition applied iteratively to each m/z × RT2
    × RT1 sub-tensor with an increasing number of components
- name: multiway
  role: R package used for multiway decomposition (PARAFAC) inside the GcDUO deconvolution
    workflow
- name: rrcov3way
  role: R package supporting robust three-way decomposition routines in the GcDUO
    deconvolution stack
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:63be3bf648a54a6bd6d3df85e40dbf0d25c3ff0ff720b71a53627f053eb95537
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
  - http://edamontology.org/topic_3855
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# tucker-congruence-component-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates blind PARAFAC-deconvolved GC × GC–MS components by iteratively increasing the component count until explained variance (R²) ceases to improve and every extracted component shows a Gaussian chromatographic profile with Tucker congruence above 0.9, as implemented in GcDUO's Module III.

## When to use

Use when performing blind (no prior component knowledge) PARAFAC deconvolution of m/z × RT2 × RT1 sub-tensors from untargeted GC × GC–TOFMS data and you must decide the correct number of components and reject non-chemical (non-Gaussian) loadings.

## When NOT to use

- When component identity/number is already known (supervised PARAFAC); the blind iterative fitting is designed for samples with no prior knowledge
- When the analysis goal is peak alignment across samples rather than per-blob deconvolution — PARAFAC2 relaxes the tri-linearity constraint for that purpose
- When input is already a peak/feature table from another tool (e.g. ChromaTOF) rather than raw CFD chromatographic data

## Inputs

- GcDUO S3 object ($data: 4D array samples × m/z × retention1 × retention2) built from CDF files via readFolderCDF with a specified modulationTime (e.g. 2.5 s) and mzRange (e.g. c(35, 440))
- ROI/peak list produced by GcDUO's procesData (signalNoiseRatio1, signalNoiseRatio2, win_size) with per-sample detected blobs

## Outputs

- Per-blob PARAFAC loadings for accepted components (Gaussian, Tucker congruence > 0.9) with per-feature spectra, retention times, and peak assignments
- deconvolution_results.rds containing per-feature spectra, retention times, and peak assignments after duplicate-feature removal and cross-sample consensus grouping
- Explained-variance (R²) and Tucker congruence diagnostics supporting the chosen component count

## How to apply

For each detected blob, extract the m/z × RT2 × RT1 sub-tensor, evaluate variation within the window, and retain only the top 5% of m/z channels for component selection. Run PARAFAC starting from a low number of components, increasing the count iteratively until the model's R² stops improving; a further requisite is that all extracted components show a Gaussian chromatographic form with Tucker congruence exceeding 0.9. Components failing the Gaussian/congruence criteria indicate either an under- or over-factored model and should trigger re-running with a different component count. After acceptance, remove duplicated features within the same sample and group features across samples by retention time and top m/z fragment using a dot-product spectral similarity threshold of 60% to compute consensus spectra. Rationale: PARAFAC is effective for deconvoluting overlapping peaks, but its open-source use is scarce due to computational cost and the need for prior knowledge of components; the iterative congruence/Gaussian test supplies that knowledge automatically.

## Related tools

- **GcDUO** (R package whose Module III wraps the blind per-sample PARAFAC deconvolution with the Tucker congruence and Gaussian validation criteria) — https://github.com/mariallr/GcDuo
- **PARAFAC** (Chemometric tri-linear decomposition applied iteratively to each m/z × RT2 × RT1 sub-tensor with an increasing number of components)
- **multiway** (R package used for multiway decomposition (PARAFAC) inside the GcDUO deconvolution workflow)
- **rrcov3way** (R package supporting robust three-way decomposition routines in the GcDUO deconvolution stack)

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- All extracted PARAFAC components display a Gaussian chromatographic profile
- Tucker congruence of every retained component exceeds 0.9
- Model explained variance (R²) no longer improves when additional components are added
- Peak areas correlate highly (R² = 0.9) with gold-standard software (ChromaTOF) peak-area measurements
- Duplicate features within a sample are removed and cross-sample grouping (dot-product similarity ≥ 60%) yields a single mean consensus spectrum per feature

## Limitations

- Computational cost of iterative PARAFAC and the necessity of prior component knowledge motivated workarounds; GcDUO addresses this but R is 'not optimized for handling large memory loads or parallel processing in multiple cores' and may need high-performance computing environments
- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted (e.g. decane was only detected when signal-to-noise was reduced from 3 to 1)
- In untargeted analysis the ROIs are unknown; inaccurate ROI selection undermines downstream deconvolution

## Evidence

- [results] iterative component-count criterion: [evidence span withheld — anchor status: not_found]
