---
name: msms-noise-ion-simulation
description: 'Use when benchmarking MS/MS library-matching similarity scores for robustness:
  you have cleaned, centroided reference spectra and need to quantify score degradation
  as artificial noise ions are added at graded levels.'
when_to_use_negative:
- When the goal is library search accuracy or false discovery rate benchmarking against
  NIST20 (434,287 spectra) rather than noise-robustness characterization of a similarity
  score
- When noise in the real data is chemical/matrix interference or signal-dependent
  noise rather than randomly added ions with uniformly sampled m/z and Poisson (lambda=1)
  intensity contributions
- When input spectra are profile-mode or lack precursor m/z metadata, since the workflow
  requires centroided peaks and precursor-ion removal above precursor_mz - 1.6 Da
tools:
- name: ms_entropy
  role: Computes entropy similarity between original and noisy spectra (calculate_entropy_similarity
    with ms2_tolerance_in_da=0.05) and provides clean_spectrum() for preprocessing
    reference spectra
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Holds spectra as peak arrays and supports uniform sampling of noise-ion m/z
    values within the instrument range
- name: SciPy
  role: Supplies the Poisson (lambda=1) distribution used to draw noise-ion intensity
    contributions
- name: Python
  role: Implements the noise simulation loop, dot product (cosine) similarity computation,
    and degradation table export to CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:4923cb45a4f005687070e0148bdcaebe06ab6f351dea6b44c7123f38e462ebc2
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# msms-noise-ion-simulation

## Summary

Simulates graded artificial noise-ion contamination on centroided NIST20 MS/MS reference spectra (m/z uniformly sampled, Poisson-distributed intensities) and quantifies how entropy similarity degrades versus dot product (cosine) similarity as noise levels increase.

## When to use

Use when benchmarking MS/MS library-matching similarity scores for robustness: you have cleaned, centroided reference spectra and need to quantify score degradation as artificial noise ions are added at graded levels.

## When NOT to use

- When the goal is library search accuracy or false discovery rate benchmarking against NIST20 (434,287 spectra) rather than noise-robustness characterization of a similarity score
- When noise in the real data is chemical/matrix interference or signal-dependent noise rather than randomly added ions with uniformly sampled m/z and Poisson (lambda=1) intensity contributions
- When input spectra are profile-mode or lack precursor m/z metadata, since the workflow requires centroided peaks and precursor-ion removal above precursor_mz - 1.6 Da

## Inputs

- Cleaned NIST20 reference MS/MS spectra (centroided, peaks <1% base peak removed, ions above precursor m/z - 1.6 Da removed, intensities normalized to sum 1)
- Graded noise-level definitions (target fractions of total relative intensity contributed by added noise)
- Precursor m/z and instrument m/z range per spectrum for sampling noise-ion locations

## Outputs

- Noisy spectra per noise level (original peaks plus uniformly sampled m/z, Poisson-distributed noise ions, re-normalized intensities)
- Entropy similarity scores and dot product (cosine) similarity scores for original vs. noisy spectrum pairs at each noise level
- Degradation table written to similarity_robustness_results.csv

## How to apply

First preprocess the NIST20 reference spectra: centroid peaks, remove low-abundance ions below 1% base peak intensity, remove ions with m/z above precursor_mz - 1.6 Da, and normalize intensities to sum = 1 (e.g. via ms_entropy.clean_spectrum). At each graded noise level, generate noise ions whose m/z values are uniformly sampled across the instrument range and whose contributions follow a Poisson distribution (lambda=1), scaling the noise so it represents a defined fraction of the total relative intensity of added noise. For each spectrum pair, re-normalize intensities and compute entropy similarity with ms_entropy.calculate_entropy_similarity using ms2_tolerance_in_da=0.05, then compute the dot product (cosine) similarity on the same pairs. The rationale is the paper's finding that entropy similarity showed marked robustness against noise ions while dot product similarity is more affected. Tabulate the score degradation of both metrics across all noise levels and write the comparison to similarity_robustness_results.csv.

## Related tools

- **ms_entropy** (Computes entropy similarity between original and noisy spectra (calculate_entropy_similarity with ms2_tolerance_in_da=0.05) and provides clean_spectrum() for preprocessing reference spectra) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Holds spectra as peak arrays and supports uniform sampling of noise-ion m/z values within the instrument range)
- **SciPy** (Supplies the Poisson (lambda=1) distribution used to draw noise-ion intensity contributions)
- **Python** (Implements the noise simulation loop, dot product (cosine) similarity computation, and degradation table export to CSV)

## Examples

```
import ms_entropy as me; score = me.calculate_entropy_similarity(cleaned_spectrum, noisy_spectrum, ms2_tolerance_in_da=0.05)
```

## Evaluation signals

- Entropy similarity degradation (score drop) must be markedly smaller than dot product similarity degradation at every noise level, consistent with the paper's NIST20 robustness finding
- After noise addition, spectrum intensities must be re-normalized to sum 1 before scoring
- Added noise must represent exactly the target fraction of total relative intensity at each graded level
- All similarity scores must fall in [0, 1]; scores for identical spectra (zero noise) must equal their maxima
- similarity_robustness_results.csv must contain one degradation row per noise level for both entropy and dot product scores

## Limitations

- Even with entropy similarity, no single similarity threshold prevents false-positive discoveries, so robust noise scores do not eliminate false positives
- Power transformations at w < 1 decrease similarity scoring performance for ion-rich spectra with high entropy values
- The simulated noise model (uniform m/z, Poisson lambda=1) is a simplified perturbation and may not reflect instrument-specific noise structures

## Evidence
