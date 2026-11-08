---
name: r-chemometric-modeling
description: Use when processing a batch of GC×GC–MS samples where a feature has been
  detected in more than one sample and you need its second-dimension elution profile
  to vary per sample while its mass spectrum is fixed to a library spectrum.
when_to_use_negative:
- The feature was detected in only one sample — a batch-constrained PARAFAC2 fit cannot
  be stabilized and GcDUO skips it in module V, preserving the single-sample data
  in module III results
- The feature was not annotated against the in-house MSP library (cosine similarity
  ≤ 0.7), so there is no library spectrum to impose as a constraint on the mass-spectrum
  mode
- Input is already a feature/area table produced by a different tool (e.g. ChromaTOF
  peak tables); this skill operates on raw GC×GC–MS arrays inside the GcDUO object,
  not on imported peak tables
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: GcDUO
  role: R package whose module V (data integration / ConsProcessing) performs the
    batch-constrained PARAFAC2 re-evaluation and quantification
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Runtime environment hosting GcDUO and the PARAFAC2 implementations (installed
    via devtools::install_github("mariallr/GcDUO") after BiocManager::install("EBImage"))
  repo: https://github.com/mariallr/GcDuo
- name: PARAFAC2
  role: Multiway decomposition algorithm used with the second-dimension mode B_kf
    free per sample and the mass-spectrum mode constrained to the library spectrum
- name: multiway
  role: Candidate R implementation for fitting the batch PARAFAC2 models
- name: rrcov3way
  role: Alternative R implementation for fitting the batch PARAFAC2 models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:03374ec8f1091cf2081d30da201691993dfe060eb6abd0d105f01d8b4c3b2bfe
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# r-chemometric-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit constrained PARAFAC2 models across all samples in an R GC×GC–MS batch to extract library-aligned elution profiles and intensities for each feature, replacing sample-by-sample deconvolution.

## When to use

Use when processing a batch of GC×GC–MS samples where a feature has been detected in more than one sample and you need its second-dimension elution profile to vary per sample while its mass spectrum is fixed to a library spectrum.

## When NOT to use

- The feature was detected in only one sample — a batch-constrained PARAFAC2 fit cannot be stabilized and GcDUO skips it in module V, preserving the single-sample data in module III results
- The feature was not annotated against the in-house MSP library (cosine similarity ≤ 0.7), so there is no library spectrum to impose as a constraint on the mass-spectrum mode
- Input is already a feature/area table produced by a different tool (e.g. ChromaTOF peak tables); this skill operates on raw GC×GC–MS arrays inside the GcDUO object, not on imported peak tables

## Inputs

- GcDuo 'DuoResults' S3 object from procesData (ind_peak, spec_ind, cor_spectra)
- Annotated feature table from duoID (id_peak with library matches)
- Consensus spectra of common features between samples (cor_spectra)
- MSP-format in-house spectral library imported via getMSP and spectramatrix
- Raw 4D GC×GC–MS array (samples x m/z fragments x retention 1 x retention 2) from CDF files

## Outputs

- Per-feature, per-sample peak-area table
- Per-feature, per-sample maximum-intensity table
- Fitted PARAFAC2 elution profiles (B_kf) for each feature
- CSV summary files of areas and intensities

## How to apply

Load the GcDUO 'DuoResults' object from the peak-picking stage together with the annotated feature table and the consensus spectra produced by duoID (annotation uses a cosine similarity match_cutoff of 0.7 against an imported MSP library, optionally with Kovats retention-index checks within RIrange = 30). For each annotated feature detected in more than one sample, assemble the chromatographic-region windows across all samples and fit a PARAFAC2 decomposition simultaneously over the batch using the multiway or rrcov3way R implementations; the second-dimension mode (B_kf) is allowed to differ per sample, while the mass-spectrum mode is constrained to the library spectrum of the annotation, so a single library compound is quantified consistently even when its elution shifts between samples. After the fit, extract the estimated elution profile and intensity, integrate to obtain peak area and maximum intensity per feature per sample, and write the per-feature, per-sample area and intensity tables as CSV. Features detected in only one sample are not further evaluated here because a batch-constrained PARAFAC2 model cannot be stabilized from a single sample, though their data is preserved in the module III results.

## Related tools

- **GcDUO** (R package whose module V (data integration / ConsProcessing) performs the batch-constrained PARAFAC2 re-evaluation and quantification) — https://github.com/mariallr/GcDuo
- **R** (Runtime environment hosting GcDUO and the PARAFAC2 implementations (installed via devtools::install_github("mariallr/GcDUO") after BiocManager::install("EBImage"))) — https://github.com/mariallr/GcDuo
- **PARAFAC2** (Multiway decomposition algorithm used with the second-dimension mode B_kf free per sample and the mass-spectrum mode constrained to the library spectrum)
- **multiway** (Candidate R implementation for fitting the batch PARAFAC2 models)
- **rrcov3way** (Alternative R implementation for fitting the batch PARAFAC2 models)

## Examples

```
finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20)
```

## Evaluation signals

- Every feature passed to module V must appear in at least two samples; single-sample features should be absent from the module V output but retained in module III (ind_peak/spec_ind)
- The fitted mass-spectrum mode of each PARAFAC2 component must reproduce the library spectrum of the corresponding annotation (high cosine similarity), confirming the constraint was honored
- Area and intensity tables must be complete (every annotated feature x every sample with a detection) and numeric, with no missing rows for batch-shared features
- Output area/intensity values should show high correlation with the gold-standard ChromaTOF peak areas (the paper reports R2 = 0.9 for GcDUO vs ChromaTOF), so regress the areas against a ChromaTOF reference as a sanity check
- The finalGcDuo object should expose the four expected slots: $peaks, $area, $intensity, $spectra, each with the feature/sample dimensions matching the annotation table

## Limitations

- GcDUO is implemented in R, which is not optimized for large memory loads or multi-core parallel processing; batches may require high-performance computing environments to reach optimal results
- Computational cost and the need for prior knowledge of components are cited as the main reasons PARAFAC-style deconvolution is scarce in open-source GC×GC software; the batch-constrained fit depends on prior library annotations
- Only features detected in more than one sample enter module V; single-sample detections are not re-evaluated, so they rely on the module III deconvolution alone
- Validation reported >89% of present peaks correctly identified across the two in-house GC×GC-TOFMS validation datasets, leaving a residual fraction of peaks without quantification

## Evidence

- [discussion] batch-constrained PARAFAC2 re-evaluation with library spectra imposed: [evidence span withheld — anchor status: not_found]
- [discussion] library spectrum constrains mass-spectrum mode: [evidence span withheld — anchor status: not_found]
- [discussion] elution profile varies between samples: [evidence span withheld — anchor status: not_found]
