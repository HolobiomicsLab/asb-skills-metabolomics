---
name: median-similarity-threshold-calibration
description: Use when you must simulate lower-quality fragmentation spectra from biological
  samples on clean reference MS/MS spectra and need an objective, similarity-based
  calibration of the noise level instead of arbitrary noise parameters.
when_to_use_negative:
- Input spectra are already low-quality biological-sample spectra — no synthetic distortion
  is needed; evaluate directly instead.
- The analysis goal is FDR estimation on real metabolomics runs rather than simulating
  degraded spectra for benchmarking.
- You need to assign confidence to structure annotations; use the confidence-score
  pipeline rather than spectrum-distortion calibration.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: COSMIC
  role: Evaluation framework whose confidence score is being tested on the distorted
    spectra produced by this calibration.
  repo: github.com/sirius-ms/sirius
- name: SIRIUS 4
  role: Processes the reference and distorted tandem MS/MS spectra to generate structure
    candidate rankings for evaluation.
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-021-01045-9
    title: High-confidence structural annotation of metabolites absent from spectral
      libraries
  source_content_hash: sha256:aecb73a653bd2e88cbfbddbe084ddc4dfab94a7d497a7d26247a2014ee130b6c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COSMIC
  - SIRIUS 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# median-similarity-threshold-calibration

## Summary

Calibrates a spectrum-distortion procedure so that injected noise reproduces a target median cosine similarity to the original reference MS/MS spectra (e.g. 0.880 for medium and 0.714 for high distortion in the COSMIC evaluation).

## When to use

Use when you must simulate lower-quality fragmentation spectra from biological samples on clean reference MS/MS spectra and need an objective, similarity-based calibration of the noise level instead of arbitrary noise parameters.

## When NOT to use

- Input spectra are already low-quality biological-sample spectra — no synthetic distortion is needed; evaluate directly instead.
- The analysis goal is FDR estimation on real metabolomics runs rather than simulating degraded spectra for benchmarking.
- You need to assign confidence to structure annotations; use the confidence-score pipeline rather than spectrum-distortion calibration.

## Inputs

- Reference tandem MS/MS spectra as peak lists (m/z, intensity pairs), e.g. from MassBank, GNPS and NIST 2017 combined dataset
- Target median cosine similarity values per distortion level (e.g. 0.880 medium, 0.714 high)

## Outputs

- Distorted reference fragmentation spectra with medium/high noise-level annotations, written to distorted_spectra.mgf
- Median cosine similarity value achieved per distortion level against the original spectra

## How to apply

Load reference tandem MS/MS spectra as peak lists (m/z, intensity pairs) and build a pooled noise-peak set from all reference spectra so that injected noise peaks retain realistic m/z and intensity distributions. For the medium distortion level, apply global and per-spectrum mass shifts, random intensity variations, and noise-peak injection, tuning the shift/intensity/noise-count parameters iteratively; repeat the same procedure with stronger parameters for the high distortion level. After each distortion round, compute the cosine similarity of every distorted spectrum against its original (using binned peak-intensity vectors) and take the median per distortion level. Accept the distortion settings only when the median cosine similarity matches the pre-specified targets — 0.880 for medium and 0.714 for high in the COSMIC evaluation — and no distorted spectrum is reduced to at most one peak. Rationale: rather than guessing noise severities, the calibration ties simulated spectral degradation to a measurable, reproducible similarity target, so that downstream confidence-score evaluation on distorted spectra is comparable to evaluation on real biological-sample spectra.

## Related tools

- **COSMIC** (Evaluation framework whose confidence score is being tested on the distorted spectra produced by this calibration.) — github.com/sirius-ms/sirius
- **SIRIUS 4** (Processes the reference and distorted tandem MS/MS spectra to generate structure candidate rankings for evaluation.) — https://github.com/sirius-ms/sirius

## Examples

```
python calibrate_noise.py --ref ref_spectra.mgf --target 0.880 --level medium --out distorted_spectra.mgf
```

## Evaluation signals

- Median cosine similarity to original spectra equals 0.880 for the medium distortion level.
- Median cosine similarity to original spectra equals 0.714 for the high distortion level.
- No distorted fragmentation spectrum contains at most one peak.
- Parameters were tuned iteratively until the similarity targets were met, not set arbitrarily.

## Limitations

- Calibration targets are specific to the COSMIC evaluation; other studies may require different similarity targets.
- Cosine similarity on binned peak-intensity vectors is a coarse proxy for real spectral degradation of biological samples.
- No guarantee that the resulting FDR behavior on distorted spectra transfers exactly to real biological datasets.

## Evidence

- [methods] medium or high levels of noise were added to the reference fragmentation spectra before evaluation: [evidence span withheld — anchor status: not_found]
- [methods] median cosine similarity equals 0.880 for medium and 0.714 for high: [evidence span withheld — anchor status: not_found]
- [discussion] Evaluations were carried out using reference measurements, as we do not know the correct answers for biological datasets.: "Evaluations were carried out using reference measurements, as we do not know the correct answers for biological datasets."
