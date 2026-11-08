---
name: spectral-constraint-optimization
description: Use when a batch of GC×GC–MS samples has already been peak-picked and
  annotated (consensus spectra available from GcDUO module III) and you need sample-wise
  peak areas/intensities that account for between-sample retention-time shifts, i.e.
  module V data integration.
when_to_use_negative:
- The feature was detected in only one sample — module V does not evaluate such features
  (their data stay in module III results).
- No annotation/consensus spectrum is available for the feature, so no library spectrum
  can be imposed as the spectral-mode constraint.
- The extracted components do not conform to a Gaussian form, which the blind PARAFAC
  requires.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: R package whose ConsProcessing function performs the constrained PARAFAC2
    re-evaluation (module V) on the GcDuoObject and DuoResults to produce areas and
    intensities per feature per sample
  repo: https://github.com/mariallr/GcDuo
- name: multiway
  role: R package used to fit the batch-constrained PARAFAC2 decomposition across
    samples
- name: rrcov3way
  role: R package used to fit the batch-constrained PARAFAC2 decomposition across
    samples
- name: devtools
  role: used to install GcDUO from GitHub
  repo: https://github.com/mariallr/GcDuo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:7511f01aa9d3a58a0d2067f6692eab1d9ae2609479dc83b72f09fb8a2e16fcd8
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
  - R
  - devtools
  - multiway
  - rrcov3way
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# spectral-constraint-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refine GC×GC–MS peak quantification by re-fitting PARAFAC2 models for each annotated feature across all samples, constraining the mass-spectrum mode to the consensus/library spectrum while allowing the elution profile to vary between samples.

## When to use

Use when a batch of GC×GC–MS samples has already been peak-picked and annotated (consensus spectra available from GcDUO module III) and you need sample-wise peak areas/intensities that account for between-sample retention-time shifts, i.e. module V data integration.

## When NOT to use

- The feature was detected in only one sample — module V does not evaluate such features (their data stay in module III results).
- No annotation/consensus spectrum is available for the feature, so no library spectrum can be imposed as the spectral-mode constraint.
- The extracted components do not conform to a Gaussian form, which the blind PARAFAC requires.

## Inputs

- Processed GcDuo results object (DuoResults) containing ind_peak, spec_ind and cor_spectra tables
- Annotated features with consensus mass spectra (from GcDUO peak annotation, cosine similarity > 0.7 against the in-house library)
- 4D raw data array (samples × m/z fragments × retention 1 × retention 2) from readFolderCDF

## Outputs

- finalGcDuo object with $peaks (summary of detected peaks), $area (area per peak and sample), $intensity (intensity per peak and sample) and $spectra
- Per-feature, per-sample peak area and maximum intensity summary tables saved as CSV files
- Estimated elution profiles (B_kf modes) for each constrained PARAFAC2 fit

## How to apply

Load the processed GcDuo results (DuoResults from procesData) and the annotated features with their consensus spectra from the earlier modules. For each annotated feature detected in more than one sample, assemble the data windows across all samples and fit a PARAFAC2 decomposition across the batch (multiway/rrcov3way in R), permitting the second-dimension elution-profile mode (B_kf) to vary between samples while constraining the mass-spectrum mode to match the library spectrum of the annotation. Extract the estimated elution profile and intensity from the fitted model and integrate to compute peak area and maximum intensity per feature per sample. Note the constraint imposed in the blind PARAFAC: all extracted components must show a Gaussian form. Assemble the per-feature, per-sample area and intensity values into summary tables and save them as CSV files; features detected in only one sample are excluded from this evaluation, though their data remain in the module III results.

## Related tools

- **GcDUO** (R package whose ConsProcessing function performs the constrained PARAFAC2 re-evaluation (module V) on the GcDuoObject and DuoResults to produce areas and intensities per feature per sample) — https://github.com/mariallr/GcDuo
- **multiway** (R package used to fit the batch-constrained PARAFAC2 decomposition across samples)
- **rrcov3way** (R package used to fit the batch-constrained PARAFAC2 decomposition across samples)
- **devtools** (used to install GcDUO from GitHub) — https://github.com/mariallr/GcDuo

## Examples

```
finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20)
```

## Evaluation signals

- The fitted PARAFAC2 spectrum mode must match the imposed library/consensus spectrum of the annotation.
- The extracted components must show a Gaussian form, as required by the blind PARAFAC.
- Peak areas and maximum intensities per feature per sample should be produced for every feature detected in more than one sample.
- Cross-validation against gold-standard software showed a high correlation (R2 = 0.9) in peak area measurements.
- More than 89% of present peaks were identified correctly in the two validation datasets.

## Limitations

- Features detected in only one sample are not further evaluated in module V; their data are preserved only in the results of module III.
- R is not optimized for handling large memory loads or parallel processing, so large batches may need high-performance computing environments.
- PARAFAC2 fitting is computationally costly, and prior knowledge of the components (the library spectra) is needed to impose the constraint.
- Small chromatographic signals with too few characteristic fragments cannot be deconvoluted and will be missing from the quantification.

## Evidence

- [results] In Module V (data integration), for each detected feature a new PARAFAC2 model is calculated, incorporating parameters such as the number of components, the chromatographic region for evaluation, and the identified compound spectra.: [evidence span withheld — anchor status: not_found]
