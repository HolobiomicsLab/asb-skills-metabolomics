---
name: dot-product-similarity-scoring
description: Use when you need a conventional cosine/dot-product similarity score
  between two centroided MS/MS spectra, e.g. as the baseline against which entropy-similarity
  robustness or library-search performance is compared.
when_to_use_negative:
- 'As the primary library-search score for noisy or ion-rich spectra: the paper shows
  dot product similarity is markedly more affected by added noise ions than entropy
  similarity'
- On spectra that have not been centroided, precursor-filtered, and intensity-normalized
  (sum = 1), since peak alignment at 0.05 Da assumes cleaned input
- When a single score cutoff is expected to prevent false positives — the paper states
  even entropy similarity cannot define such a threshold, and dot product fares worse
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NumPy
  role: Vectorized computation of the dot product (cosine) over aligned intensity
    vectors
- name: SciPy
  role: Statistics utilities (e.g. Poisson sampling) for generating graded noise ions
    in the robustness comparison
- name: ms_entropy
  role: Reference implementation used to compute entropy similarity on the same spectrum
    pairs for baseline comparison against dot product scores
  repo: https://github.com/YuanyueLi/MSEntropy
- name: Python
  role: Glue language for spectrum loading, cleaning, scoring, and CSV output
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:16c18e23cb996a860d287086ba067ff00b7005335e6f48a57c9220bcf8c3ed4b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - NumPy
  - Python
  - SciPy
  - ms_entropy
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# dot-product-similarity-scoring

## Summary

Computes the classic dot product (cosine) similarity between two centroided, intensity-normalized MS/MS peak lists — the conventional library-matching baseline that Li et al. benchmark spectral entropy similarity against under added Poisson noise.

## When to use

Use when you need a conventional cosine/dot-product similarity score between two centroided MS/MS spectra, e.g. as the baseline against which entropy-similarity robustness or library-search performance is compared.

## When NOT to use

- As the primary library-search score for noisy or ion-rich spectra: the paper shows dot product similarity is markedly more affected by added noise ions than entropy similarity
- On spectra that have not been centroided, precursor-filtered, and intensity-normalized (sum = 1), since peak alignment at 0.05 Da assumes cleaned input
- When a single score cutoff is expected to prevent false positives — the paper states even entropy similarity cannot define such a threshold, and dot product fares worse

## Inputs

- Centroided MS/MS peak lists as [[m/z, intensity], ...] arrays (query and reference), intensities normalized to sum 1
- Cleaned NIST20 reference spectra with low-abundance (<1% base peak) and precursor-region ions (m/z > precursor_mz - 1.6 Da) removed
- Optional: noisy spectrum variants with graded Poisson-distributed (lambda=1) noise ions added

## Outputs

- A single dot product (cosine) similarity score in [0, 1] per spectrum pair
- Degradation table (score drop per noise level) written to similarity_robustness_results.csv when used in the robustness benchmark

## How to apply

Preprocess both spectra before scoring: centroid peaks, remove low-abundance ions below 1% base-peak intensity, remove ions with m/z above precursor_mz - 1.6 Da (shown on NIST20 and Massbank.us to enhance spectral identification), and normalize intensities to sum to 1. For each spectrum pair, match peaks within an MS2 tolerance of 0.05 Da and compute the dot product of the aligned intensity vectors — equivalently the cosine score for unweighted spectra. In robustness experiments, generate graded noise ions whose m/z values are uniformly sampled in the instrument range and whose contributions follow a Poisson distribution (lambda=1), scaled to a defined fraction of the total relative intensity of added noise, then re-normalize and re-score. Tabulate the score drop of the dot product across all noise levels alongside entropy similarity (e.g. from ms_entropy.calculate_entropy_similarity) to expose the dot product's greater sensitivity to noise ions.

## Related tools

- **NumPy** (Vectorized computation of the dot product (cosine) over aligned intensity vectors)
- **SciPy** (Statistics utilities (e.g. Poisson sampling) for generating graded noise ions in the robustness comparison)
- **ms_entropy** (Reference implementation used to compute entropy similarity on the same spectrum pairs for baseline comparison against dot product scores) — https://github.com/YuanyueLi/MSEntropy
- **Python** (Glue language for spectrum loading, cleaning, scoring, and CSV output) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
import numpy as np; s = np.dot(i1, i2) / np.sqrt(np.dot(i1, i1) * np.dot(i2, i2))
```

## Evaluation signals

- Score must lie in [0, 1]; identical cleaned spectra must yield a score of 1.0
- Peak matching must use the 0.05 Da MS2 tolerance stated in the workflow
- As noise-ion fraction increases, the dot product must degrade (drop) more than the entropy similarity on the same pairs — this asymmetry is the expected result
- Output table must cover all graded noise levels and be written to similarity_robustness_results.csv
- Spectra must have been renormalized (sum to 1) after noise addition before scoring

## Limitations

- Dot product similarity is markedly more affected by added noise ions than entropy similarity, so its scores under noisy conditions overstate true spectral relatedness
- No single similarity threshold can prevent false-positive discoveries even with entropy scores, so dot product thresholds are even less reliable
- Performance conclusions are grounded in NIST20/Massbank.us high-resolution small-molecule data; ENTACT benchmark data were not publicly available to the authors

## Evidence

- [abstract] Dot product is the conventional baseline whose degradation under noise quantifies its fragility: "Entropy similarity outperformed 42 alternative similarity algorithms, including dot product similarity, when searching 434,287 spectra against the high-quality NIST20 library"
- [discussion] Noise-addition design for the robustness comparison that dot product scores are part of: "noise ions were artificially added to NIST20 mass spectra on the basis of total relative intensity of added noise"
- [discussion] Expected comparative outcome when scoring noisy pairs: "The entropy similarity score showed marked robustness against noise ions"
- [methods] Precursor-ion removal preprocessing needed before scoring: "eliminating ions with m/z values higher than the precursor ion's m/z - 1.6 considerably enhances the spectral identification"
- [discussion] Spectral cleaning and normalization convention applied before similarity computation: "the peaks from MS/MS spectrum were centroided first, then all ion intensities were adjusted to have sum = 1"
- [readme] README statement positioning dot product as the outperformed similarity metric: [evidence span withheld — anchor status: not_found]
