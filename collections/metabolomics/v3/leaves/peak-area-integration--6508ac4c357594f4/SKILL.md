---
name: peak-area-integration
description: Use when GC × GC–MS deconvoluted features must be quantified across all
  samples — the annotated peaks from GcDUO's module III need area/intensity values,
  not just detection and annotation.
when_to_use_negative:
- Input data has not yet been peak-picked and annotated (module I–III incomplete)
  — running module V on raw GcDuoObject alone is invalid.
- Only a single-sample chromatogram must be quantified — GcDUO's batch-constrained
  PARAFAC2 re-evaluation is designed for features detected across multiple samples.
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0078
tools:
- name: GcDUO
  role: Provides the ConsProcessing function that performs the batch-constrained PARAFAC2
    re-evaluation (module V) and the area/intensity integration.
  repo: https://github.com/mariallr/GcDuo
- name: PARAFAC2
  role: Algorithm used for the constrained decomposition; elution-profile mode is
    allowed to vary between samples while the spectrum mode is fixed to the library
    spectrum.
- name: R
  role: Platform in which GcDUO is implemented; the skill is invoked through R function
    calls (GcDuo::ConsProcessing).
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:491b4d271aaff7267187ab6d898e3d820df641a5dfd347772e44fad15d02ca19
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
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0078
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# peak-area-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-feature peak areas and maximum intensities for each sample in a batch GC × GC–MS run by re-fitting PARAFAC2 models constrained by library spectra, and assembles them into summary tables (CSV).

## When to use

Use when GC × GC–MS deconvoluted features must be quantified across all samples — the annotated peaks from GcDUO's module III need area/intensity values, not just detection and annotation.

## When NOT to use

- Input data has not yet been peak-picked and annotated (module I–III incomplete) — running module V on raw GcDuoObject alone is invalid.
- Only a single-sample chromatogram must be quantified — GcDUO's batch-constrained PARAFAC2 re-evaluation is designed for features detected across multiple samples.

## Inputs

- GcDuoObject (4D raw data array: samples × m/z fragments × RT1 × RT2)
- DuoResults object with $ind_peak, $spec_ind, $cor_spectra
- Annotated features ($id_peak) with consensus spectra from module III
- Library spectra (from getMSP/spectramatrix) used as PARAFAC2 constraints
- win_width: an average width for the peaks (default 20 modulations)

## Outputs

- finalGcDuo$peaks: summary table of detected peaks
- finalGcDuo$area: matrix of peak areas per feature per sample
- finalGcDuo$intensity: matrix of max intensity per feature per sample
- finalGcDuo$spectra: spectra per feature per sample
- CSV files of area and intensity tables

## How to apply

Load the processed GcDuo results (DuoResults from procesData) and the annotated features with consensus spectra. For each annotated feature detected in more than one sample, assemble the data windows across all samples and fit a PARAFAC2 decomposition simultaneously, permitting the second-dimension elution-profile mode (B_kf) to vary between samples while constraining the mass-spectrum mode to match the library spectrum of the annotation. Integrate the estimated elution profile to compute peak area and maximum intensity per feature per sample. Assemble the per-feature, per-sample values into summary tables and save as CSV. Features detected in only one sample are not further evaluated here — their data is preserved in the module III results.

## Related tools

- **GcDUO** (Provides the ConsProcessing function that performs the batch-constrained PARAFAC2 re-evaluation (module V) and the area/intensity integration.) — https://github.com/mariallr/GcDuo
- **PARAFAC2** (Algorithm used for the constrained decomposition; elution-profile mode is allowed to vary between samples while the spectrum mode is fixed to the library spectrum.)
- **R** (Platform in which GcDUO is implemented; the skill is invoked through R function calls (GcDuo::ConsProcessing).)

## Examples

```
finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20)
```

## Evaluation signals

- Per-feature area and intensity matrices have one column per sample and one row per annotated feature — no rows should be missing for features annotated in ≥2 samples.
- Peak areas from GcDUO should correlate with ChromaTOF gold-standard values at approximately R² = 0.9 for validated datasets.
- Spectra of the re-fitted components must match the library spectra used as constraints (cosine similarity > 0.7 was required during annotation).
- All extracted components should show a Gaussian form; non-Gaussian elution profiles indicate a failed PARAFAC2 fit.
- Features detected in only one sample must be excluded from the area/intensity tables (they remain in module III results).

## Limitations

- Computationally costly: PARAFAC2 is computationally intensive and may require high-performance computing environments for large batches.
- R is not optimized for handling large memory loads or parallel processing in multiple cores — batch size may be limited by available RAM.
- Signals with too few characteristic fragments (small chromatographic signals) fail to deconvolute and yield no area/intensity values.
- Features detected in only one sample are not evaluated, so their quantitative information is lost from the final table.

## Evidence

- [results] Module V (data integration), for each detected feature a new PARAFAC2 model is calculated: [evidence span withheld — anchor status: not_found]
- [results] batch PARAFAC2 constrains the mass-spectrum mode to the library spectrum: [evidence span withheld — anchor status: not_found]
