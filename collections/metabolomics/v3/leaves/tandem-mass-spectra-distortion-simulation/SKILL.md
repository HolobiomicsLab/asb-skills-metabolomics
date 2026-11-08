---
name: tandem-mass-spectra-distortion-simulation
description: Use when evaluating a structure-annotation confidence score (e.g. COSMIC)
  on reference MS/MS spectra and you need to emulate the lower quality of biological-sample
  spectra at controlled distortion levels.
when_to_use_negative:
- The input spectra are already real biological-sample spectra with authentic low
  quality — no simulation of noise is needed
- The goal is scoring confidence on pristine library spectra without evaluating robustness
  to degradation
- The task is spectral library search or de-replication rather than calibration of
  confidence-score evaluation under distortion
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: Framework for processing the reference tandem MS/MS spectra that are distorted
    and used for confidence-score evaluation
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Confidence-score workflow whose evaluation on reference datasets consumes
    the medium/high distorted spectra produced by this skill
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:f0e27d260289398a00420e4a194ea1a492167e83fc2b951777b0e4d078e1ffc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# tandem-mass-spectra-distortion-simulation

## Summary

Simulates the lower quality of biological-sample tandem MS/MS by adding medium or high levels of distortion (mass shifts, intensity variations, noise-peak injection) to reference fragmentation spectra, calibrated so the median cosine similarity to the originals matches 0.880 (medium) and 0.714 (high). This lets a confidence score such as COSMIC be evaluated under realistic spectrum degradation.

## When to use

Use when evaluating a structure-annotation confidence score (e.g. COSMIC) on reference MS/MS spectra and you need to emulate the lower quality of biological-sample spectra at controlled distortion levels.

## When NOT to use

- The input spectra are already real biological-sample spectra with authentic low quality — no simulation of noise is needed
- The goal is scoring confidence on pristine library spectra without evaluating robustness to degradation
- The task is spectral library search or de-replication rather than calibration of confidence-score evaluation under distortion

## Inputs

- Reference tandem MS/MS spectra as peak lists (pairs of m/z and intensity values)
- Distortion-level settings (mass shift, intensity variation, noise-peak count parameters) for medium and high levels
- Target median cosine similarity values: 0.880 (medium) and 0.714 (high)

## Outputs

- distorted_spectra.mgf containing the distorted spectra annotated with distortion level
- Per-level median cosine similarity between distorted and original spectra (0.880 medium, 0.714 high)
- Pooled noise-peak set built from peaks across all reference spectra

## How to apply

Load the reference tandem MS/MS spectra as peak lists of m/z and intensity pairs, then build a pooled noise-peak set by collecting peaks across all reference spectra so injected noise peaks retain realistic m/z and intensity distributions. For the medium distortion level, apply global and per-spectrum mass shifts, random intensity variations, and noise-peak injection, tuning the shift/intensity/noise-count parameters iteratively; repeat the same distortion procedure with stronger parameters for the high level. After each round, compute the cosine similarity between every distorted spectrum and its original (using binned peak-intensity vectors) and take the median per distortion level. Adjust parameters until the median cosine similarity reaches 0.880 for medium and 0.714 for high, then write the distorted spectra with level annotations to distorted_spectra.mgf. Throughout, discard any spectrum reduced to at most one peak — mirroring the paper's rule that compounds were dropped from all libraries when a fragmentation spectrum with at most one peak resulted at any collision energy and noise level.

## Related tools

- **SIRIUS 4** (Framework for processing the reference tandem MS/MS spectra that are distorted and used for confidence-score evaluation) — https://github.com/sirius-ms/sirius
- **COSMIC** (Confidence-score workflow whose evaluation on reference datasets consumes the medium/high distorted spectra produced by this skill) — https://github.com/sirius-ms/sirius

## Evaluation signals

- Median cosine similarity between distorted and original spectra (binned peak-intensity vectors) equals 0.880 for the medium level and 0.714 for the high level
- No distorted spectrum is reduced to at most one peak
- Injected noise peaks are drawn from the pooled peak set so their m/z and intensity distributions match the reference spectra
- Output file distorted_spectra.mgf carries level annotations for every distorted spectrum

## Limitations

- Evaluations were carried out using reference measurements, as the correct answers for biological datasets are unknown — the simulated distortion is a proxy, not measured noise
- A compound was discarded from all libraries if a fragmentation spectrum with at most one peak resulted for at least one collision energy and noise level, so heavily distorted spectra may be excluded
- Calibration relies on median cosine similarity targets (0.880/0.714), which are practical guidance only and do not guarantee comparable FDR behavior on biological annotations

## Evidence

- [discussion] Discard compound from all libraries if any collision energy/noise level yields a spectrum with at most one peak: "we discarded the compound from all libraries if a fragmentation spectrum with, at most, one peak resulted for at least one collision energy and noise level"
- [discussion] Evaluations use reference measurements because biological ground truth is unavailable: "Evaluations were carried out using reference measurements, as we do not know the correct answers for biological datasets."
- [readme] SIRIUS is the software framework for analyzing LC-MS/MS data of metabolites: [evidence span withheld — anchor status: external_doc]
