---
name: msms-intensity-weighting
description: Use when a centroided, normalized MS/MS peak list must be intensity-weighted
  prior to entropy similarity computation and the spectrum's entropy (or ion richness)
  is unknown or low (S < 3).
when_to_use_negative:
- 'The spectrum has high entropy (S >= 3) / is ion-rich: applying power transformations
  with w < 1 decreases similarity scoring performance for such spectra, so w = 1 should
  be used instead'
- 'Peaks are raw and uncentroided: preprocessing (centroiding, normalization to sum
  = 1) via ms_entropy.clean_spectrum is required before entropy calculation and weighting'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms_entropy
  role: Provides calculate_spectral_entropy (to derive S with clean_spectrum=False)
    and apply_weight_to_intensity (to transform intensities as I^w and renormalize
    to sum = 1)
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Loads and manipulates the float32 [m/z, intensity] peak arrays
- name: Python
  role: Implementation environment for the weighting workflow
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:0c689f2ef298cc1f1b923bacb85b10771b14627d909e0027514a1b283b5afae6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - NumPy
  - Python
  - ms_entropy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# msms-intensity-weighting

## Summary

Applies an entropy-dependent power transformation to MS/MS ion intensities — I_new = I^w with w = 0.25 + S*0.25 for spectra with spectral entropy S < 3, and w = 1 for S >= 3 — before similarity scoring, boosting low-abundance ions in ion-poor small-molecule spectra.

## When to use

Use when a centroided, normalized MS/MS peak list must be intensity-weighted prior to entropy similarity computation and the spectrum's entropy (or ion richness) is unknown or low (S < 3).

## When NOT to use

- The spectrum has high entropy (S >= 3) / is ion-rich: applying power transformations with w < 1 decreases similarity scoring performance for such spectra, so w = 1 should be used instead
- Peaks are raw and uncentroided: preprocessing (centroiding, normalization to sum = 1) via ms_entropy.clean_spectrum is required before entropy calculation and weighting

## Inputs

- Cleaned, normalized MS/MS peak array ([[m/z, intensity], ...] as float32) with intensities summing to 1 (e.g. spectrum_peaks.npy)
- Spectral entropy S of the spectrum (computed with ms_entropy.calculate_spectral_entropy, clean_spectrum=False)

## Outputs

- Weighted peak list (m/z, weighted intensity) where weighted intensities sum to 1, e.g. weighted_peaks.csv
- Applied weight exponent w (= 0.25 + 0.25*S for S < 3, capped at 1 for S >= 3)

## How to apply

Load the cleaned, normalized peak array ([[m/z, intensity], ...] as float32) and compute the spectral entropy S with ms_entropy.calculate_spectral_entropy using clean_spectrum=False, since peaks are already centroided and normalized to sum to 1. If S < 3, derive the exponent w = 0.25 + S*0.25; if S >= 3, keep w = 1 because power transformations with w < 1 decrease similarity scoring performance for ion-rich, high-entropy spectra. Transform each intensity as I^w and renormalize the weighted intensities so they sum to 1 — this is what ms_entropy.apply_weight_to_intensity does. This weighting boosts low-abundance ions in low-entropy, ion-poor small-molecule spectra, improving entropy similarity matching. Save the weighted peak list (m/z, weighted intensity) and verify the intensity column sums to 1 within floating-point tolerance.

## Related tools

- **ms_entropy** (Provides calculate_spectral_entropy (to derive S with clean_spectrum=False) and apply_weight_to_intensity (to transform intensities as I^w and renormalize to sum = 1)) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Loads and manipulates the float32 [m/z, intensity] peak arrays)
- **Python** (Implementation environment for the weighting workflow) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
import numpy as np, ms_entropy as me; peaks = np.load('spectrum_peaks.npy'); S = me.calculate_spectral_entropy(peaks, clean_spectrum=False); w = 0.25 + 0.25*S if S < 3 else 1.0; weighted = me.apply_weight_to_intensity(peaks, weight=w)
```

## Evaluation signals

- The weighted intensity column sums to 1 within floating-point tolerance
- The applied exponent follows w = 0.25 + 0.25*S for S < 3 and is capped at w = 1 for S >= 3
- calculate_spectral_entropy was called with clean_spectrum=False on already-centroided, normalized peaks
- Low-abundance ions gain relative weight relative to the original normalized intensities (I^w with w < 1 compresses dynamic range)
- Output peak list retains the original m/z values paired with transformed intensities (e.g. weighted_peaks.csv schema)

## Limitations

- Power transformations with w < 1 decrease similarity scoring performance for ion-rich spectra with high entropy values, hence the S >= 3 cap at w = 1
- Weighting is only meaningful after centroiding and normalization (intensities adjusted to sum = 1); applying it to raw peak lists is invalid
- Even with entropy similarity, no single similarity threshold prevents false-positive discoveries

## Evidence

- [discussion] Entropy-dependent intensity weighting rule: [evidence span withheld — anchor status: not_found]
