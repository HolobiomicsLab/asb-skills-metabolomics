---
name: multimode-elution-profile-alignment
description: Use when processing a batch of GC×GC–MS samples in GcDUO where a feature's
  retention-2 elution profile shifts between samples and you need per-sample peak
  areas/intensities without forcing rigid tri-linearity.
when_to_use_negative:
- The feature is detected in only one sample — GcDUO skips Module V evaluation for
  such features and preserves their data from module III instead
- You need a rigid tri-linear decomposition where all modes must be identical across
  samples — plain PARAFAC, not PARAFAC2, is the appropriate model
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: R package whose Module V performs the constrained batch PARAFAC2 re-evaluation
    via the ConsProcessing function
  repo: https://github.com/mariallr/GcDuo
- name: PARAFAC2
  role: Chemometric model that relaxes tri-linearity so the second-dimension elution
    profile can vary per sample while the mass-spectrum mode is constrained to the
    library spectrum
- name: R
  role: Implementation platform for GcDUO and the PARAFAC2 fitting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:6c9a51078bf04ae7207a3436acc6df58a71892ce53059fb83ceb72a8a890d839
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# multimode-elution-profile-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns GC×GC–MS features across a batch by refitting PARAFAC2 models per feature with the second-dimension elution-profile mode free to vary between samples, while imposing the annotated library mass spectrum as a constraint, yielding comparable peak areas and intensities per sample.

## When to use

Use when processing a batch of GC×GC–MS samples in GcDUO where a feature's retention-2 elution profile shifts between samples and you need per-sample peak areas/intensities without forcing rigid tri-linearity.

## When NOT to use

- The feature is detected in only one sample — GcDUO skips Module V evaluation for such features and preserves their data from module III instead
- You need a rigid tri-linear decomposition where all modes must be identical across samples — plain PARAFAC, not PARAFAC2, is the appropriate model

## Inputs

- GcDuoObject (4D CDF-derived array: samples × m/z × RT1 × RT2)
- DuoResults object with annotated features (id_peak annotations, consensus spectra cor_spectra)
- Imported spectral library (libs_dic + spectra_matrix) used to constrain mass-spectrum mode

## Outputs

- finalGcDuo$peaks: summary table of detected features
- finalGcDuo$area: peak area for each feature and sample
- finalGcDuo$intensity: maximum intensity for each feature and sample
- finalGcDuo$spectra: fitted spectra for each feature and sample

## How to apply

In GcDUO's Module V (ConsProcessing/data integration), take the processed DuoResults object and the annotated features whose consensus spectra were derived in Module IV; features detected in only one sample are excluded from further evaluation because their profile cannot be aligned across the batch. For each remaining feature, assemble the data windows across all samples and fit a new PARAFAC2 decomposition simultaneously on all samples, letting the second-dimension elution-profile mode (B_kf) differ per sample while constraining the mass-spectrum mode to match the library spectrum of the annotation. Choose win_width as an average peak width (e.g. 20 modulations in the README example) to bound the chromatographic region evaluated. Extract the fitted elution profile and intensity, integrate to compute peak area and maximum intensity per feature per sample, and assemble the per-feature, per-sample area and intensity tables. PARAFAC2 is chosen over PARAFAC because it relaxes the tri-linearity constraint, enabling alignment between samples with slightly shifted elution profiles.

## Related tools

- **GcDUO** (R package whose Module V performs the constrained batch PARAFAC2 re-evaluation via the ConsProcessing function) — https://github.com/mariallr/GcDuo
- **PARAFAC2** (Chemometric model that relaxes tri-linearity so the second-dimension elution profile can vary per sample while the mass-spectrum mode is constrained to the library spectrum)
- **R** (Implementation platform for GcDUO and the PARAFAC2 fitting)

## Examples

```
finalGcDuo <- GcDuo::ConsProcessing(GcDuoObject, DuoResults, lib, win_width = 20)
```

## Evaluation signals

- finalGcDuo$area and finalGcDuo$intensity are sample-by-feature matrices whose row/column counts match the annotated features retained after module III/IV filtering
- Every annotated feature appearing in more than one sample yields a non-NA area; single-sample features are absent from module V output but present in module III results
- The fitted spectra per sample should remain consistent with the library spectrum used as the constraint (spectra mode is imposed, not free)
- Peak areas validated against gold-standard software (ChromaTOF) show high correlation (R2 = 0.9), so a sanity check is comparing a few features' areas against a reference deconvolution
- All extracted components should still conform to a Gaussian form, consistent with the Gaussian-requisite filter applied in the blind PARAFAC stage

## Limitations

- Features detected in only one sample are not further evaluated in module V — they are preserved in module III results but excluded from batch quantification
- GcDUO is implemented in R, which is not optimized for large memory loads or multi-core parallel processing; batch PARAFAC2 fits may require high-performance computing environments
- Peak-area agreement with ChromaTOF is high (R2 = 0.9) but not identical, so absolute values may differ from gold-standard software

## Evidence

- [results] Module V: batch constrained PARAFAC2 re-evaluation: [evidence span withheld — anchor status: not_found]
