---
name: cosine-similarity-spectral-matching
description: Use when evaluating a structure-annotation confidence score (e.g. COSMIC)
  on clean reference spectra whose biological use case involves noisier spectra, or
  when quantifying spectral distortion/merging spectra per precursor feature by cosine
  distance.
when_to_use_negative:
- Input spectra are already noisy biological-sample spectra — the noise-addition procedure
  exists to simulate such spectra from clean reference measurements
- A fragmentation spectrum has at most one peak — such spectra were removed in this
  workflow
- The goal is establishing stereochemistry or distinguishing small structure modifications
  — MS/MS data alone is insufficient here
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS 4
  role: Processes the LC–MS/MS runs; its feature pipeline merges per-precursor fragmentation
    spectra using cosine-distance agglomerative clustering, and the distorted reference
    spectra feed the COSMIC confidence-score evaluation
  repo: https://github.com/sirius-ms/sirius
- name: COSMIC
  role: Confidence score whose evaluation on reference datasets depends on spectra
    distorted to medium/high noise levels via cosine-similarity calibration
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:fec751204273c5db42dde2ea3ebf1effda19b8986c3c74b94151f31712a8328f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - COSMIC
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cosine-similarity-spectral-matching

## Summary

Uses cosine similarity between binned peak-intensity vectors of fragmentation spectra for two purposes in this paper: merging fragmentation spectra assigned to the same precursor feature, and quantifying how much simulated noise (medium/high distortion) degrades reference MS/MS spectra to emulate lower-quality biological-sample spectra.

## When to use

Use when evaluating a structure-annotation confidence score (e.g. COSMIC) on clean reference spectra whose biological use case involves noisier spectra, or when quantifying spectral distortion/merging spectra per precursor feature by cosine distance.

## When NOT to use

- Input spectra are already noisy biological-sample spectra — the noise-addition procedure exists to simulate such spectra from clean reference measurements
- A fragmentation spectrum has at most one peak — such spectra were removed in this workflow
- The goal is establishing stereochemistry or distinguishing small structure modifications — MS/MS data alone is insufficient here

## Inputs

- Reference tandem MS/MS spectra as peak lists (pairs of m/z and intensity values)
- Fragmentation spectra assigned to a common precursor ion / feature for per-feature merging

## Outputs

- Distorted spectra with level annotations (medium/high) written to distorted_spectra.mgf
- Median cosine similarity per distortion level (target 0.880 for medium, 0.714 for high)

## How to apply

Load the reference tandem MS/MS spectra as peak lists of m/z and intensity pairs. Build a pooled noise-peak set from peaks across all reference spectra so injected noise peaks retain realistic m/z and intensity distributions. For the medium distortion level, apply global and per-spectrum mass shifts, random intensity variations, and noise-peak injection, tuning shift/intensity/noise-count parameters iteratively; repeat with stronger parameters for the high level. For each distorted spectrum, compute cosine similarity against its original spectrum using binned peak-intensity vectors and take the median per level; adjust parameters until the median equals 0.880 (medium) and 0.714 (high), and ensure no distorted spectrum is reduced to at most one peak. Write the distorted spectra with level annotations to distorted_spectra.mgf. The same cosine metric also underpins merging: fragmentation spectra assigned to the same feature (precursor ion) are merged via agglomerative clustering based on cosine distance.

## Related tools

- **SIRIUS 4** (Processes the LC–MS/MS runs; its feature pipeline merges per-precursor fragmentation spectra using cosine-distance agglomerative clustering, and the distorted reference spectra feed the COSMIC confidence-score evaluation) — https://github.com/sirius-ms/sirius
- **COSMIC** (Confidence score whose evaluation on reference datasets depends on spectra distorted to medium/high noise levels via cosine-similarity calibration)

## Examples

```
from scipy.spatial.distance import cosine; sim = 1 - cosine(binned_original, binned_distorted)  # binned peak-intensity vectors; tune params until median sim == 0.880 (medium) / 0.714 (high)
```

## Evaluation signals

- Median cosine similarity of distorted spectra to originals equals 0.880 for the medium level and 0.714 for the high level
- No distorted spectrum is reduced to at most one peak
- Injected noise peaks are drawn from the pooled peak set across all reference spectra, preserving realistic m/z and intensity distributions
- Distorted spectra are annotated with their distortion level in the output MGF
- Per-feature spectrum merging (if applied) uses agglomerative clustering with cosine distance

## Limitations

- Confidence score must not be mistaken as probability the annotation is correct; this is impossible by design
- Accurate FDR estimation from small molecule fragmentation spectra expected to remain highly challenging
- Evaluations rely on reference measurements because no ground truth exists for biological datasets; similar FDR cannot be guaranteed for biological annotations
- Small structure modifications are hard or impossible to tell apart using MS/MS data alone

## Evidence

- [methods] Noise simulation targets: [evidence span withheld — anchor status: not_found]
- [discussion] Cosine-distance merging: "Fragmentation spectra assigned to the same feature (precursor ion) are merged using an agglomerative clustering algorithm based on cosine distance."
- [discussion] Minimum-peak filter: "we removed fragmentation spectra with, at most, one peak"
- [discussion] Reference-based evaluation: "Evaluations were carried out using reference measurements, as we do not know the correct answers for biological datasets."
