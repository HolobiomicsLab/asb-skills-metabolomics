---
name: spectral-entropy-similarity-scoring
description: Use when matching experimental MS/MS spectra against spectral libraries
  (NIST20, MassBank/MoNA, GNPS), when comparing similarity scoring algorithms for
  compound identification, or when quantifying noise robustness of spectral similarity
  scores.
when_to_use_negative:
- Input spectra have not been centroided/cleaned — applying similarity directly to
  raw profile peaks without clean_spectrum degrades matching (ions above precursor_mz
  - 1.6 must be removed first)
- The goal is isomer-level discrimination requiring structure-aware scoring — the
  plain entropy similarity score cannot separate highly similar structures; the one-bond-difference
  variant is needed instead
- Spectra are ion-rich with high entropy values and power transformations at w < 1
  are being applied — the paper reports such transformations decrease similarity scoring
  performance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms_entropy
  role: Python package providing clean_spectrum, calculate_entropy_similarity, calculate_unweighted_entropy_similarity,
    apply_weight_to_intensity and read_one_spectrum used for all scoring steps
  repo: https://github.com/YuanyueLi/MSEntropy
- name: MSEntropy
  role: Successor package integrating spectral entropy, entropy similarity and Flash
    entropy search; recommended over the legacy SpectralEntropy repo
  repo: https://github.com/YuanyueLi/MSEntropy
- name: NumPy
  role: Handles peak arrays ([m/z, intensity] float32 2D arrays) for spectrum representation
    and manipulation
- name: SciPy
  role: Alternative source of the entropy function (scipy.stats.entropy) if clean_spectrum
    is skipped
- name: Python 3.7.9
  role: Implementation language for the 43 similarity algorithms benchmarked in Supplementary
    Note 1
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:a9182d9448a5ed70d82c1d4354282a6cc533dbfb9e8dbc00a7c688d220bc7a91
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - MSEntropy
  - NumPy
  - Python
  - Python 3.7.9
  - R 3.6.3
  - SciPy
  - ms_entropy
  - multiprocessing
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 24
---

# spectral-entropy-similarity-scoring

## Summary

Computes entropy similarity between MS/MS spectra using the ms_entropy package (after clean_spectrum preprocessing), outperforming dot-product/cosine similarity for library matching on NIST20 and MassBank of North America spectra, with FDR < 10% at score 0.75.

## When to use

Use when matching experimental MS/MS spectra against spectral libraries (NIST20, MassBank/MoNA, GNPS), when comparing similarity scoring algorithms for compound identification, or when quantifying noise robustness of spectral similarity scores.

## When NOT to use

- Input spectra have not been centroided/cleaned — applying similarity directly to raw profile peaks without clean_spectrum degrades matching (ions above precursor_mz - 1.6 must be removed first)
- The goal is isomer-level discrimination requiring structure-aware scoring — the plain entropy similarity score cannot separate highly similar structures; the one-bond-difference variant is needed instead
- Spectra are ion-rich with high entropy values and power transformations at w < 1 are being applied — the paper reports such transformations decrease similarity scoring performance

## Inputs

- MS/MS spectra as dictionaries with 'precursor_mz' (float) and 'peaks' (2D array of [m/z, intensity]) via ms_entropy.read_one_spectrum
- NIST20 high-resolution library MS/MS spectra (434,287 spectra in the benchmark)
- Experimental spectra from MassBank of North America (Massbank.us) and GNPS library spectra

## Outputs

- Entropy similarity scores (and dot-product baseline scores) per query-library spectrum pair
- Similarity-score threshold versus FDR curves (e.g. fdr_threshold_analysis.csv)
- Noise-robustness degradation tables (e.g. similarity_robustness_results.csv)

## How to apply

First pre-process every spectrum with ms_entropy.clean_spectrum: remove empty peaks (m/z <= 0 or intensity <= 0), remove ions above precursor_mz - 1.6 Da, centroid peaks within +/- min_ms2_difference_in_da = 0.05 Da, drop peaks below noise_threshold = 0.01 of the base peak, retain the top max_peak_num peaks, and normalize intensities to sum 1. Then compute entropy similarity with ms_entropy.calculate_entropy_similarity at ms2 tolerance 0.05 Da; the score derives from Shannon spectral entropy weighting of ion intensities rather than raw intensity products. For baseline comparison, compute the dot product (cosine) similarity on the same cleaned peak pairs. Key decision points: use unweighted variants (calculate_unweighted_entropy_similarity) when testing power transforms, and sweep score thresholds to build FDR curves rather than fixing a single cutoff, since no single threshold prevents all false-positive discoveries.

## Related tools

- **ms_entropy** (Python package providing clean_spectrum, calculate_entropy_similarity, calculate_unweighted_entropy_similarity, apply_weight_to_intensity and read_one_spectrum used for all scoring steps) — https://github.com/YuanyueLi/MSEntropy
- **MSEntropy** (Successor package integrating spectral entropy, entropy similarity and Flash entropy search; recommended over the legacy SpectralEntropy repo) — https://github.com/YuanyueLi/MSEntropy
- **NumPy** (Handles peak arrays ([m/z, intensity] float32 2D arrays) for spectrum representation and manipulation)
- **SciPy** (Alternative source of the entropy function (scipy.stats.entropy) if clean_spectrum is skipped)
- **Python 3.7.9** (Implementation language for the 43 similarity algorithms benchmarked in Supplementary Note 1) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
import numpy as np, ms_entropy as me; peaks_a = np.array([[69.071, 7.917962], [86.066, 1.021589]]); score = me.calculate_entropy_similarity(peaks_a, peaks_b, ms2_tolerance_in_da=0.05)
```

## Evaluation signals

- FDR curve must show false discovery rates below 10% at entropy similarity score 0.75 on the 37,299 natural-product spectra (row nearest 0.75 shows FDR < 0.10)
- Noise-robustness check: entropy similarity score degradation under added Poisson-distributed noise ions (lambda = 1, uniform m/z sampling) must be smaller than dot-product degradation at every noise level
- All computed FDR values must lie between 0 and 1
- Cleaned spectra must have intensities summing to 1, no peaks above precursor_mz - 1.6 Da, and no peaks below 1% of base-peak intensity
- Benchmark expectation: entropy similarity should outperform all 42 alternative algorithms on FDR/TPR threshold curves against the NIST20 library

## Limitations

- No single similarity threshold can be defined that would prevent all false-positive discoveries — threshold sweeps are required per dataset
- Power transformations at w < 1 decrease similarity scoring performance for ion-rich spectra with high entropy values
- ENTACT study data for benchmarking are not publicly available
- No version numbers stated for ms_entropy or numpy; matching at 0.05 Da tolerance is fixed by the paper's protocol and may not suit other instruments

## Evidence

- [readme] MSEntropy package recommendation: [evidence span withheld — anchor status: external_doc]
