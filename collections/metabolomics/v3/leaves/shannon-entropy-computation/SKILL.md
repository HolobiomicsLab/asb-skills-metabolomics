---
name: shannon-entropy-computation
description: Use when a raw centroided MS/MS spectrum (peak list of m/z, intensity
  pairs) needs to be summarized by a single complexity value, e.g. as the basis for
  entropy-based similarity scoring or noise-robust spectral comparison.
when_to_use_negative:
- The goal is to compare two spectra rather than characterize one — use entropy similarity
  (calculate_entropy_similarity) instead, which weights matched peaks by entropy.
- Input spectra are already processed into feature tables or similarity matrices,
  with no raw peak lists available to centroid and normalize.
- Spectra are deliberately kept unweighted for unweighted dot-product-style comparison
  (calculate_unweighted_entropy_similarity), where entropy weighting would change
  the scoring.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ms_entropy
  role: Python package providing read_one_spectrum, clean_spectrum, and calculate_spectral_entropy
    used to read, clean, normalize, and compute the entropy
  repo: https://github.com/YuanyueLi/MSEntropy
- name: SciPy
  role: 'Alternative entropy computation: scipy.stats.entropy applied directly to
    the normalized intensity column'
- name: NumPy
  role: Holds the cleaned spectrum as a 2D (m/z, intensity) array and supports normalization
    of intensities to sum to 1
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:955316b445ada826a534f145a4f920571508f61b2060e65452585ab84e5f8926
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# shannon-entropy-computation

## Summary

Computes MS/MS spectral entropy S = -sum(I_p ln I_p), a Shannon-entropy-derived measure of spectrum complexity, by treating the normalized peak-intensity distribution of a centroided tandem mass spectrum as a probability distribution.

## When to use

Use when a raw centroided MS/MS spectrum (peak list of m/z, intensity pairs) needs to be summarized by a single complexity value, e.g. as the basis for entropy-based similarity scoring or noise-robust spectral comparison.

## When NOT to use

- The goal is to compare two spectra rather than characterize one — use entropy similarity (calculate_entropy_similarity) instead, which weights matched peaks by entropy.
- Input spectra are already processed into feature tables or similarity matrices, with no raw peak lists available to centroid and normalize.
- Spectra are deliberately kept unweighted for unweighted dot-product-style comparison (calculate_unweighted_entropy_similarity), where entropy weighting would change the scoring.

## Inputs

- Single MS/MS spectrum as a peak list of (m/z, intensity) pairs (NumPy 2D array or dictionary from ms_entropy.read_one_spectrum, e.g. from .mgf, .msp, .mzML files)
- Precursor m/z and precursor_ions_removal_da (default 1.6) for removing residual precursor ions
- Cleaning parameters: min_ms2_difference_in_da (0.05), noise_threshold (0.01), max_peak_num

## Outputs

- Scalar spectral entropy value S (natural-log based) in the range [0, ln(number of retained peaks)]
- spectral_entropy.json recording the computed entropy and the cleaned, normalized spectrum

## How to apply

Read the spectrum with ms_entropy.read_one_spectrum, then pre-process with ms_entropy.clean_spectrum: remove empty peaks (m/z <= 0 or intensity <= 0), remove ions with m/z above precursor_mz - 1.6 Da, merge peaks within +/- min_ms2_difference_in_da = 0.05 Da and sort by m/z, drop peaks below noise_threshold (0.01 of base-peak intensity was used in the paper), keep the top max_peak_num peaks, and finally normalize intensities so they sum to 1 — this turns the intensity vector into a probability distribution I_p. Compute S = -sum(I_p ln I_p) with ms_entropy.calculate_spectral_entropy (set clean_spectrum=False because peaks are already cleaned), or equivalently with scipy.stats.entropy on the normalized intensity column. The rationale is that spectral entropy quantifies the uniformity of the intensity distribution and underpins entropy similarity, which the paper showed outperforms dot-product similarity for small-molecule MS/MS identification.

## Related tools

- **ms_entropy** (Python package providing read_one_spectrum, clean_spectrum, and calculate_spectral_entropy used to read, clean, normalize, and compute the entropy) — https://github.com/YuanyueLi/MSEntropy
- **SciPy** (Alternative entropy computation: scipy.stats.entropy applied directly to the normalized intensity column)
- **NumPy** (Holds the cleaned spectrum as a 2D (m/z, intensity) array and supports normalization of intensities to sum to 1)

## Examples

```
import ms_entropy as me; cleaned = me.clean_spectrum(peaks, min_ms2_difference_in_da=0.05, noise_threshold=0.01); S = me.calculate_spectral_entropy(cleaned, clean_spectrum=False)
```

## Evaluation signals

- Normalized intensities sum to 1 after the clean_spectrum step-6 normalization.
- Computed entropy lies within 0 <= S <= ln(number of retained peaks); S approaches 0 for a single dominant peak and approaches ln(n) for a uniform n-peak spectrum.
- The same S value is obtained from scipy.stats.entropy on the intensity column as from ms_entropy.calculate_spectral_entropy(clean_spectrum=False).
- No peaks with m/z > precursor_mz - 1.6 Da remain, and merged peaks are separated by more than 0.05 Da after centroiding.
- Peaks below 1% of base-peak intensity (noise_threshold=0.01) have been removed from the cleaned spectrum.

## Limitations

- Spectral entropy is only meaningful after centroiding and intensity normalization; applying it to unnormalized raw intensities gives an invalid result.
- The paper notes that power transformations of intensities at weight w < 1 decrease similarity-scoring performance for ion-rich spectra with high entropy values, so entropy-heavy spectra should not be down-weighted.
- calculate_spectral_entropy normally performs its own centroiding step; if peaks are pre-cleaned, clean_spectrum=False must be set to avoid double cleaning.

## Evidence
