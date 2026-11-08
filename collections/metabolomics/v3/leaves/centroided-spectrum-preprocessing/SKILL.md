---
name: centroided-spectrum-preprocessing
description: Use when MS/MS peaks are already centroided and normalized (sum = 1)
  and a low-entropy, ion-poor spectrum (S < 3) needs intensity re-weighting prior
  to entropy similarity or library matching.
when_to_use_negative:
- 'The spectrum is ion-rich with high entropy (S >= 3): w stays at 1, and applying
  a power transformation with w < 1 there decreases similarity scoring performance.'
- 'Peaks are not yet centroided or normalized: run ms_entropy.clean_spectrum() first
  (remove empty peaks, remove peaks above precursor_mz - precursor_ions_removal_da,
  merge peaks within +/- min_ms2_difference_in_da, noise threshold, top max_peak_num,
  normalize to sum 1) instead of computing entropy directly.'
- The downstream scoring uses unweighted similarity (e.g. calculate_unweighted_entropy_similarity
  or classic dot product), where entropy weighting is not part of the method.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms_entropy
  role: Provides calculate_spectral_entropy (clean_spectrum=False for pre-centroided
    peaks) and apply_weight_to_intensity to transform intensities as I^w and renormalize
    to sum 1
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Loads and holds the float32 [[m/z, intensity], ...] peak array before and
    after weighting
- name: MSEntropy
  role: Package integrating spectral entropy and entropy similarity functions recommended
    over the legacy SpectralEntropy code
  repo: https://github.com/YuanyueLi/MSEntropy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:e9b9477c5d7e94efae4cb160493898c598ab7ea565390dd83275a1f23ee1502f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - MSEntropy
  - NumPy
  - Python
  - ms_entropy
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# centroided-spectrum-preprocessing

## Summary

Entropy-dependent intensity weighting of already centroided, normalized MS/MS peak lists: spectral entropy S is computed and ion intensities are transformed as I^w with w = 0.25 + 0.25*S for low-entropy (S < 3) spectra, boosting low-abundance ions before entropy similarity scoring.

## When to use

Use when MS/MS peaks are already centroided and normalized (sum = 1) and a low-entropy, ion-poor spectrum (S < 3) needs intensity re-weighting prior to entropy similarity or library matching.

## When NOT to use

- The spectrum is ion-rich with high entropy (S >= 3): w stays at 1, and applying a power transformation with w < 1 there decreases similarity scoring performance.
- Peaks are not yet centroided or normalized: run ms_entropy.clean_spectrum() first (remove empty peaks, remove peaks above precursor_mz - precursor_ions_removal_da, merge peaks within +/- min_ms2_difference_in_da, noise threshold, top max_peak_num, normalize to sum 1) instead of computing entropy directly.
- The downstream scoring uses unweighted similarity (e.g. calculate_unweighted_entropy_similarity or classic dot product), where entropy weighting is not part of the method.

## Inputs

- Cleaned, centroided, normalized peak array [[m/z, intensity], ...] as float32 (e.g. spectrum_peaks.npy loaded with NumPy)

## Outputs

- Weighted peak list (m/z, weighted intensity) with intensities summing to 1, saved as weighted_peaks.csv

## How to apply

Load the cleaned peak array ([[m/z, intensity], ...] as float32) — e.g. from spectrum_peaks.npy — with NumPy; because peaks are already centroided and normalized to sum = 1, call ms_entropy.calculate_spectral_entropy with clean_spectrum=False. Derive the exponent w = 0.25 + S*0.25 when S < 3, and set w = 1 when S >= 3. Apply ms_entropy.apply_weight_to_intensity to transform each intensity as I^w and renormalize so the weighted intensities sum to 1. Write the weighted peak list (m/z, weighted intensity) to weighted_peaks.csv. The rationale (Li et al., Nat Methods 2021) is that w < 1 flattens intensity differences in ion-poor small-molecule spectra, while high-entropy ion-rich spectra must keep w = 1 because power transformations there decrease similarity scoring performance.

## Related tools

- **ms_entropy** (Provides calculate_spectral_entropy (clean_spectrum=False for pre-centroided peaks) and apply_weight_to_intensity to transform intensities as I^w and renormalize to sum 1) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Loads and holds the float32 [[m/z, intensity], ...] peak array before and after weighting)
- **MSEntropy** (Package integrating spectral entropy and entropy similarity functions recommended over the legacy SpectralEntropy code) — https://github.com/YuanyueLi/MSEntropy

## Examples

```
S = me.calculate_spectral_entropy(peaks, clean_spectrum=False); w = min(0.25 + 0.25*S, 1.0); weighted = me.apply_weight_to_intensity(peaks, weight=w)
```

## Evaluation signals

- The weighted intensity column sums to 1 within floating-point tolerance.
- The applied exponent follows w = 0.25 + 0.25*S for S < 3 and is capped at w = 1 for S >= 3.
- For w < 1, every intensity increases (I^w > I) and relative intensities flatten, boosting low-abundance ions; verify low-abundance peaks gain relative weight.
- Spectral entropy S is computed on the pre-normalized peak list with clean_spectrum=False, since peaks are already centroided.
- Output CSV schema matches (m/z, weighted intensity) with the same number of peaks as the input array.

## Limitations

- Power transformations with w < 1 decrease similarity scoring performance on ion-rich spectra with high entropy values, so weighting must be skipped (w = 1) when S >= 3.
- No version numbers are stated for ms_entropy or numpy in the paper's code examples, so reproducibility across package versions is not guaranteed.
- The card context reports no computed numeric result values, so expected weighted scores cannot be validated against published numbers.

## Evidence

- [methods] for spectra with entropy S < 3, intensities are transformed as I_new = I^w with w = 0.25 + S*0.25, while w = 1 is used for S >= 3: ".. autofunction:: ms_entropy.apply_weight_to_intensity"
- [discussion] the weighted intensities are then renormalized to sum to 1: "the peaks from MS/MS spectrum were centroided first, then all ion intensities were adjusted to have sum = 1"
- [discussion] if such power transformations are used for ion-rich spectra with high entropy values, similarity scoring performance is decreased: "if such power transformations are used for ion-rich spectra with high entropy values, similarity scoring performance is decreased."
- [readme] Spectral entropy is a useful property to measure the complexity of a spectrum: [evidence span withheld — anchor status: external_doc]
