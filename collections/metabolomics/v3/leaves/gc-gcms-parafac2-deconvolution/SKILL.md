---
name: gc-gcms-parafac2-deconvolution
description: Use when raw GC×GC–MS data (CDF files) contains overlapping or co-eluting
  peaks that must be resolved into pure component spectra and elution profiles across
  a batch of samples, especially when elution profiles shift between samples.
when_to_use_negative:
- Input is already a deconvoluted peak table (e.g. exported from ChromaTOF) — the
  skill is for raw 4D data, not post-processing of existing results.
- The data is 1D GC–MS without a second-dimension modulation; PARAFAC2's sample-wise
  elution-profile flexibility targets the 2D retention structure.
- 'A peak was detected in only one sample: module V skips it and only its module III
  data is preserved, so batch-constrained re-evaluation is not applicable.'
edam_operation: http://edamontology.org/operation_3647
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: GcDUO
  role: R package implementing the PARAFAC/PARAFAC2 deconvolution modules (procesData,
    ConsProcessing) for GC×GC–MS data
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Implementation platform for GcDUO and its multiway decomposition routines
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:1c908ab307b4fbdc147dd6320aba4a2f7e068effd0d3b8b9bfecae178e5f6fce
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
  - PARAFAC2
  - R
  - multiway
  - rrcov3way
  edam_operation: http://edamontology.org/operation_3647
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# gc-gcms-parafac2-deconvolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deconvolutes overlapping GC×GC–MS peaks by fitting PARAFAC or PARAFAC2 multiway decompositions within a region of interest, using PARAFAC2 when retention alignment across samples is needed. This is the core chemometric engine of GcDUO's modules III and V, producing pure spectra and elution profiles from CDF-derived 4D arrays.

## When to use

Use when raw GC×GC–MS data (CDF files) contains overlapping or co-eluting peaks that must be resolved into pure component spectra and elution profiles across a batch of samples, especially when elution profiles shift between samples.

## When NOT to use

- Input is already a deconvoluted peak table (e.g. exported from ChromaTOF) — the skill is for raw 4D data, not post-processing of existing results.
- The data is 1D GC–MS without a second-dimension modulation; PARAFAC2's sample-wise elution-profile flexibility targets the 2D retention structure.
- A peak was detected in only one sample: module V skips it and only its module III data is preserved, so batch-constrained re-evaluation is not applicable.

## Inputs

- Raw GC×GC–MS CDF files from a batch of samples
- Modulation time (seconds) of the 2D separation
- Acquired m/z fragment range (e.g. c(35, 440))
- In Module V: consensus spectra and feature annotations from prior GcDUO modules

## Outputs

- Pure component spectra and elution profiles per feature (spec_ind)
- Consensus spectra of features common between samples (cor_spectra)
- Peak area and maximum intensity per feature per sample (area, intensity tables)
- Feature table (ind_peak) with peak locations sample-by-sample

## How to apply

Import CDF files with GcDUO::readFolderCDF specifying modulationTime and mzRange (e.g. c(35, 440)) to build a 4D array (samples × m/z × retention1 × retention2). In module II, segment the data into rolling windows (win_size, minimum 3 modulations) and delineate ROIs via an inverse watershed algorithm, retaining only peaks with adequate signal-to-noise (signalNoiseRatio1/2, e.g. 50) and Gaussian form; also restrict deconvolution to the top 5% most variable m/z channels. Fit blind PARAFAC to each ROI to separate co-eluting components, or PARAFAC2 when the second-dimension elution profiles vary between samples, since PARAFAC2 relaxes the tri-linearity constraint to allow sample-wise alignment; impose the requirement that all extracted components show Gaussian form. In module V, for features detected in more than one sample, refit PARAFAC2 across all samples with the annotated library spectrum constrained as the mass-spectrum mode, then integrate the elution profile to compute peak area and maximum intensity per feature per sample.

## Related tools

- **GcDUO** (R package implementing the PARAFAC/PARAFAC2 deconvolution modules (procesData, ConsProcessing) for GC×GC–MS data) — https://github.com/mariallr/GcDuo
- **R** (Implementation platform for GcDUO and its multiway decomposition routines) — https://github.com/mariallr/GcDuo

## Examples

```
DuoResults <- GcDuo::procesData(GcDuoObject, signalNoiseRatio1 = 50, signalNoiseRatio2 = 50, win_size = 3)
```

## Evaluation signals

- All extracted components from the blind PARAFAC must conform to a Gaussian chromatographic form — a component that is non-Gaussian indicates a wrong number of components or poor ROI segmentation.
- Deconvoluted peak areas should correlate with the gold-standard software (ChromaTOF); GcDUO achieved R² = 0.9 in peak area measurements.
- In module V, the fitted PARAFAC2 mass-spectrum mode must match the annotated library spectrum (the constraint imposed); cosine similarity to the library should be ≥ 0.7 to have returned an annotation.
- Every feature evaluated in module V must have been detected in more than one sample; single-sample features must remain in module III results only.
- The resulting finalGcDuo object must contain $peaks, $area, $intensity, and $spectra with dimensions consistent with the feature table.

## Limitations

- GcDUO is implemented in R, an environment not optimized for large memory loads or parallel multi-core processing, so batch PARAFAC2 fitting may need high-performance computing.
- Features detected in only one sample are not further evaluated in module V.
- Computational cost of PARAFAC on large GC×GC–MS datasets is high, which is why the ROI segmentation and top-5% m/z prefiltering exist.

## Evidence

- [results] Module V refits PARAFAC2 per feature across samples, constraining spectrum mode to the library spectrum: [evidence span withheld — anchor status: not_found]
