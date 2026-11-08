---
name: spectral-entropy-calculation
description: Use when MS/MS peak arrays ([m/z, intensity] pairs, intensities summing
  to 1) need a complexity measure or when scoring query-vs-library spectral matches
  for small-molecule identification, e.g. against NIST20 or MassBank of North America.
when_to_use_negative:
- 'Input spectra are ion-rich with high entropy (S >= 3): do not apply power weighting
  with w < 1, as the paper shows this decreases similarity scoring performance'
- 'The analysis goal is to guarantee zero false positives: the paper states no single
  similarity threshold can prevent false-positive discoveries, even with entropy similarity
  scores'
- Spectra are profile (uncentroided) raw data and the workflow assumes clean_spectrum=False;
  you must preprocess with ms_entropy.clean_spectrum first
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ms_entropy
  role: Python package providing calculate_spectral_entropy, calculate_entropy_similarity,
    calculate_unweighted_entropy_similarity, and apply_weight_to_intensity used for
    all entropy computations
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Holds peak arrays as np.ndarray([[m/z, intensity], ...], dtype=np.float32)
    passed into ms_entropy functions
- name: SciPy
  role: 'Optional: entropy function from scipy.stats can substitute when clean_spectrum
    is skipped'
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:4a0b115263f95f3c19d720413dcdea08d2d43c24b57cbbc383b80daf58b6954c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - NumPy
  - Python
  - SciPy
  - ms_entropy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.47368421052631576
n_claims: 19
---

# spectral-entropy-calculation

## Summary

Compute Shannon-inspired spectral entropy from centroided, intensity-normalized MS/MS peak lists and use it as the basis for entropy similarity scoring of small-molecule spectra via the ms_entropy package, outperforming dot-product similarity for compound identification.

## When to use

Use when MS/MS peak arrays ([m/z, intensity] pairs, intensities summing to 1) need a complexity measure or when scoring query-vs-library spectral matches for small-molecule identification, e.g. against NIST20 or MassBank of North America.

## When NOT to use

- Input spectra are ion-rich with high entropy (S >= 3): do not apply power weighting with w < 1, as the paper shows this decreases similarity scoring performance
- The analysis goal is to guarantee zero false positives: the paper states no single similarity threshold can prevent false-positive discoveries, even with entropy similarity scores
- Spectra are profile (uncentroided) raw data and the workflow assumes clean_spectrum=False; you must preprocess with ms_entropy.clean_spectrum first

## Inputs

- Centroided MS/MS peak list as numpy array [[m/z, intensity], ...] with dtype float32 and intensities normalized to sum = 1
- Paired query and library spectra (each with precursor_mz and peaks dict) for similarity scoring
- Optional precursor_mz for upstream cleaning (remove peaks above precursor_mz - 1.6 Da)

## Outputs

- Spectral entropy S (float) of one spectrum
- Entropy similarity score in [0, 1] with components S_A, S_B, S_AB (e.g. saved to similarity_score.json)
- Entropy-weighted peak list (m/z, weighted intensity) summing to 1 (e.g. weighted_peaks.csv)

## How to apply

Load peaks as a float32 NumPy array of [m/z, intensity] pairs; if peaks are already centroided and normalized (sum = 1), call ms_entropy.calculate_spectral_entropy with clean_spectrum=False, otherwise let the function centroid first and then compute entropy. To score two spectra A and B, match ions within 0.05 Da tolerance (after removing <1% base-peak ions) into merged spectrum AB, renormalize to sum 1, and compute unweighted entropy similarity as 1 - (2*S_AB - S_A - S_B)/ln(4) (equation 2). For low-entropy (S < 3) ion-poor spectra, boost low-abundance ions by weighting intensities as I^w with w = 0.25 + 0.25*S via apply_weight_to_intensity before similarity; keep w = 1 for ion-rich, high-entropy spectra since power transformations with w < 1 decrease performance there. Optionally cross-check results with ms_entropy.calculate_unweighted_entropy_similarity (ms2_tolerance_in_da = 0.05) or scipy.stats.entropy for the entropy term alone.

## Related tools

- **ms_entropy** (Python package providing calculate_spectral_entropy, calculate_entropy_similarity, calculate_unweighted_entropy_similarity, and apply_weight_to_intensity used for all entropy computations) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Holds peak arrays as np.ndarray([[m/z, intensity], ...], dtype=np.float32) passed into ms_entropy functions)
- **SciPy** (Optional: entropy function from scipy.stats can substitute when clean_spectrum is skipped)

## Examples

```
import ms_entropy as me; S = me.calculate_spectral_entropy(peaks, clean_spectrum=False)
```

## Evaluation signals

- Spectral entropy S lies in [0, ln(n)] where n is the number of peaks; S = 0 for a single-peak spectrum
- Intensity columns after weighting/normalization sum to 1 within floating-point tolerance
- Entropy similarity scores are floats in [0, 1] and match recomputation via ms_entropy.calculate_unweighted_entropy_similarity with ms2_tolerance_in_da = 0.05
- Applied weight exponent follows w = 0.25 + 0.25*S for S < 3 and is capped at w = 1 for S >= 3
- Ions are matched within the stated 0.05 Da tolerance and low-abundance ions (<1% base peak) are removed upstream

## Limitations

- Even with entropy similarity scores, no single similarity threshold prevents false-positive discoveries
- Power transformations with w < 1 decrease performance on ion-rich, high-entropy spectra
- ms_entropy and numpy version numbers are not stated in the paper; results depend on package version (Flash entropy search in MSEntropy improved speed without compromising accuracy)
- Performance claims were benchmarked on NIST20 (434,287 searched spectra), MassBank of North America (142,903 spectra), and GNPS (33,019 spectra); other domains are untested here

## Evidence

- [readme] Spectral entropy is an useful property to measure the complexity of a spectrum: [evidence span withheld — anchor status: not_found]
