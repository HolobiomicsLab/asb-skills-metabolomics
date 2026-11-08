---
name: spectral-library-benchmarking
description: Use when quantifying how robust a spectral similarity score (e.g., entropy
  similarity vs. dot product) is to added noise ions during MS/MS library matching,
  or when designing score-degradation benchmarks on NIST20-style centroided spectra.
when_to_use_negative:
- When the goal is end-to-end compound identification benchmarking (FDR/AUC on real
  experimental query spectra) rather than controlled noise-perturbation robustness
  testing
- When spectra are not centroided or not pre-cleaned (raw profile peaks, unnormalized
  intensities) — the noise model and similarity computations assume the cleaned, sum-normalized
  spectral representation
- When evaluating search/indexing speed (e.g., Flash Entropy Search or DynamicEntropySearch)
  rather than score robustness under noise
tools:
- name: ms_entropy
  role: Computes entropy similarity between original and noise-perturbed spectra via
    calculate_entropy_similarity with ms2_tolerance_in_da=0.05
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Arrays for peak lists and generation of uniformly sampled noise ion m/z values
    with Poisson(lambda=1) intensity contributions
- name: SciPy
  role: Alternative entropy computation via scipy.stats.entropy and numerical support
    for the benchmark comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:423bd71be759a717e8a5fa06ae8d399f57a4ec2a90a34da049819bc83a5dd328
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
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# spectral-library-benchmarking

## Summary

A controlled benchmarking procedure for MS/MS spectral similarity scores in which graded levels of artificial noise ions (Poisson-weighted, uniformly sampled m/z) are added to cleaned NIST20 reference spectra, and the degradation of entropy similarity versus dot product (cosine) similarity is tabulated.

## When to use

Use when quantifying how robust a spectral similarity score (e.g., entropy similarity vs. dot product) is to added noise ions during MS/MS library matching, or when designing score-degradation benchmarks on NIST20-style centroided spectra.

## When NOT to use

- When the goal is end-to-end compound identification benchmarking (FDR/AUC on real experimental query spectra) rather than controlled noise-perturbation robustness testing
- When spectra are not centroided or not pre-cleaned (raw profile peaks, unnormalized intensities) — the noise model and similarity computations assume the cleaned, sum-normalized spectral representation
- When evaluating search/indexing speed (e.g., Flash Entropy Search or DynamicEntropySearch) rather than score robustness under noise

## Inputs

- Cleaned, centroided NIST20 reference MS/MS spectra (peaks <1% base peak removed; peaks above precursor m/z - 1.6 Da removed; intensities normalized to sum 1)
- Noise-level grid (defined fractions of total relative intensity to be replaced by noise ions)
- Precursor m/z values for each reference spectrum

## Outputs

- Per-noise-level similarity scores for entropy similarity and dot product (cosine) similarity
- Degradation (score drop) table comparing entropy vs. dot product similarity across noise levels
- similarity_robustness_results.csv

## How to apply

Start from cleaned, centroided NIST20 reference spectra: remove low-abundance ions below 1% base peak intensity, remove peaks above precursor m/z - 1.6 Da, and normalize intensities to sum to 1. At each graded noise level, generate noise ions whose m/z values are uniformly sampled across the instrument range and whose intensity contributions follow a Poisson distribution (lambda=1), scaling the noise to a defined fraction of the total relative intensity of added noise. For each noise level, re-normalize intensities and compute entropy similarity between the original and noisy spectra using ms_entropy.calculate_entropy_similarity with ms2_tolerance_in_da=0.05; compute the dot product (cosine) similarity for the same spectrum pairs. Tabulate the score drop (degradation) of entropy similarity versus dot product across all noise levels and write the comparison table to similarity_robustness_results.csv. The benchmark is correct if entropy similarity shows markedly smaller degradation than dot product similarity as noise levels increase, consistent with the reported robustness of the entropy score.

## Related tools

- **ms_entropy** (Computes entropy similarity between original and noise-perturbed spectra via calculate_entropy_similarity with ms2_tolerance_in_da=0.05) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Arrays for peak lists and generation of uniformly sampled noise ion m/z values with Poisson(lambda=1) intensity contributions)
- **SciPy** (Alternative entropy computation via scipy.stats.entropy and numerical support for the benchmark comparison)

## Examples

```
import ms_entropy as me; sim = me.calculate_entropy_similarity(peaks_original, peaks_noisy, ms2_tolerance_in_da=0.05)
```

## Evaluation signals

- At every noise level, the entropy similarity degradation must be markedly smaller than the dot product similarity degradation on the same spectrum pairs
- Both similarity scores must fall within [0, 1] for all original/noisy spectrum pairs
- Intensities of noisy spectra must be re-normalized to sum to 1 before similarity computation, matching the paper's centroid-and-normalize convention
- Noise ions must have m/z values uniformly sampled within the instrument range with Poisson(lambda=1) contributions, scaled to the specified fraction of total relative intensity
- similarity_robustness_results.csv must contain both scores tabulated per noise level with no missing pairs

## Limitations

- Even with entropy similarity scores, no single similarity threshold can be defined that would prevent false-positive discoveries
- Power transformations at w < 1 decrease similarity scoring performance for ion-rich spectra with high entropy values
- Low similarity thresholds (e.g., noise_threshold=0.01 filtering) and 0.05 Da matching tolerance are specific to this benchmark setup and may not transfer to other instruments or libraries

## Evidence

- [readme] README rationale for entropy vs dot product: [evidence span withheld — anchor status: not_found]
