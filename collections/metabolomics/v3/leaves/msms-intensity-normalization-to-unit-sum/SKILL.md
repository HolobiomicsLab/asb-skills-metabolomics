---
name: msms-intensity-normalization-to-unit-sum
description: Use when a centroided MS/MS spectrum has been cleaned (peaks merged within
  +/- 0.05 Da, noise-filtered) and you need intensities as a probability distribution
  for spectral entropy or entropy-similarity calculations.
when_to_use_negative:
- The spectrum is still profile-mode or has not been centroided/merged — normalization
  alone does not fix redundant peaks within +/- 0.05 Da
- You need raw relative intensities (e.g. % base peak) for reporting or dot-product
  similarity on unweighted intensities rather than entropy-based comparison
- Intensities have already been entropy-weighted via ms_entropy.apply_weight_to_intensity
  for entropy similarity — re-normalizing would break the weighting
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: ms_entropy
  role: Provides clean_spectrum() whose step 6 normalizes intensities to sum to 1,
    and calculate_spectral_entropy() computed on the normalized distribution
  repo: https://github.com/YuanyueLi/MSEntropy
- name: SciPy
  role: 'Alternative: scipy.stats.entropy can directly compute the entropy on the
    normalized intensity column, skipping clean_spectrum'
- name: NumPy
  role: Holds the peak array ([[m/z, intensity], ...]) passed to ms_entropy functions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:e04304b12fe69151abb3457c4b8cb192e6c62f347172920a84493d2fc2652f80
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# msms-intensity-normalization-to-unit-sum

## Summary

Normalizes cleaned, centroided MS/MS peak intensities so they sum to 1, converting the peak-intensity distribution into a probability distribution as step 6 of the ms_entropy clean_spectrum procedure. This is the prerequisite for computing Shannon-style spectral entropy S = -sum(I_p ln I_p).

## When to use

Use when a centroided MS/MS spectrum has been cleaned (peaks merged within +/- 0.05 Da, noise-filtered) and you need intensities as a probability distribution for spectral entropy or entropy-similarity calculations.

## When NOT to use

- The spectrum is still profile-mode or has not been centroided/merged — normalization alone does not fix redundant peaks within +/- 0.05 Da
- You need raw relative intensities (e.g. % base peak) for reporting or dot-product similarity on unweighted intensities rather than entropy-based comparison
- Intensities have already been entropy-weighted via ms_entropy.apply_weight_to_intensity for entropy similarity — re-normalizing would break the weighting

## Inputs

- Centroided MS/MS peak list (m/z, intensity pairs, e.g. numpy array from ms_entropy.read_one_spectrum)
- Spectrum metadata: precursor_mz (float) for precursor-ion removal
- clean_spectrum parameters: min_ms2_difference_in_da=0.05, noise_threshold=0.01, max_peak_num

## Outputs

- Normalized peak list with intensities summing to 1 (same m/z values, rescaled intensities)
- Basis input for spectral entropy S in [0, ln(number of retained peaks)] saved to spectral_entropy.json

## How to apply

First preprocess the spectrum with ms_entropy.clean_spectrum: remove empty peaks (m/z <= 0 or intensity <= 0), remove peaks above precursor_mz - precursor_ions_removal_da, centroid by merging peaks within +/- min_ms2_difference_in_da = 0.05 Da and sorting by m/z, remove peaks below noise_threshold * maximum intensity (noise_threshold=0.01), and retain only the top max_peak_num peaks. Then, as the final step, normalize the retained peak intensities so that all intensities sum to 1. This converts the intensity vector into a probability distribution over which Shannon entropy is well defined; the downstream entropy is computed via ms_entropy.calculate_spectral_entropy with clean_spectrum=False, or equivalently with scipy.stats.entropy on the normalized intensity column. Rationale: the paper's spectral entropy treats the spectrum's peak-intensity distribution as the input quantity, so unnormalized intensities would yield an invalid entropy scale.

## Related tools

- **ms_entropy** (Provides clean_spectrum() whose step 6 normalizes intensities to sum to 1, and calculate_spectral_entropy() computed on the normalized distribution) — https://github.com/YuanyueLi/MSEntropy
- **SciPy** (Alternative: scipy.stats.entropy can directly compute the entropy on the normalized intensity column, skipping clean_spectrum)
- **NumPy** (Holds the peak array ([[m/z, intensity], ...]) passed to ms_entropy functions)

## Examples

```
import numpy as np, ms_entropy as me; cleaned = me.clean_spectrum(peaks, min_ms2_difference_in_da=0.05, noise_threshold=0.01); S = me.calculate_spectral_entropy(cleaned, clean_spectrum=False)
```

## Evaluation signals

- Sum of normalized intensities equals 1 (within floating-point tolerance) after normalization
- Resulting spectral entropy lies in [0, ln(number of retained peaks)] before saving to spectral_entropy.json
- m/z values are unchanged by normalization and sorted ascending (peaks merged within +/- 0.05 Da and sorted by m/z)
- Entropy of a single-peak (fully dominated) spectrum approaches 0; entropy approaches ln(N) when N retained peaks have equal intensities
- Applying clean_spectrum with the same parameters to the same input is deterministic and reproducible

## Limitations

- Entropy is sensitive to the cleaning parameters: low-abundance ions <1% base peak intensity are removed and ions matched at 0.05 Da tolerance, so different thresholds change the distribution and thus the entropy value
- Normalization to unit sum makes the entropy a pure complexity measure of relative intensities; power transformations at w < 1 applied to ion-rich, high-entropy spectra decrease similarity scoring performance
- Version numbers for ms_entropy and numpy are not stated in the article's code examples, so exact numeric outputs are not reproducible from the text alone

## Evidence

- [readme] Spectral entropy as Shannon-entropy-based complexity measure: [evidence span withheld — anchor status: not_found]
