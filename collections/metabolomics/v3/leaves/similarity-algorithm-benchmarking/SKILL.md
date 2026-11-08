---
name: similarity-algorithm-benchmarking
description: Use when you must compare spectral similarity metrics head-to-head on
  a spectral library search — e.g., deciding whether entropy similarity beats dot
  product or 40+ alternatives on NIST20/MassBank-style MS/MS spectra.
when_to_use_negative:
- Input is raw LC-MS/MS instrument data needing acquisition or peak picking — this
  skill assumes pre-acquired, precursor-m/z-annotated peak lists
- You need a single universal score cutoff that prevents false positives — the article
  finds no single similarity threshold prevents false-positive discoveries, so per-threshold
  FDR/TPR curves, not a fixed cutoff, are the output
- Benchmarking in-silico-only libraries without experimental ground truth, where true/false
  identification labels cannot be assigned
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: ms_entropy
  role: Provides clean_spectrum for spectral pre-processing and calculate_entropy_similarity
    for the entropy similarity score being benchmarked
  repo: https://github.com/YuanyueLi/MSEntropy
- name: MSEntropy
  role: Current package integrating spectral entropy, entropy similarity, and related
    functions; recommended over the original SpectralEntropy repo for the most recent
    version
  repo: https://github.com/YuanyueLi/MSEntropy
- name: Python 3.7.9
  role: Language in which the 43 similarity algorithms were implemented (Supplementary
    Note 1) for benchmarking
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: R 3.6.3
  role: Used for the benchmarking analysis code alongside Python
- name: NumPy
  role: Array representation of spectra (m/z, intensity pairs) passed to ms_entropy
    functions
- name: SciPy
  role: scipy.stats entropy function can be used to skip the clean_spectrum step
- name: multiprocessing
  role: Parallelizes similarity computations across the large spectrum sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:98c18c00204e21f368e991e8a066faa875d54a00961add7c5fe7010d45716200
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - MSEntropy
  - NumPy
  - Python 3.7.9
  - R 3.6.3
  - SciPy
  - ms_entropy
  - multiprocessing
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# similarity-algorithm-benchmarking

## Summary

Benchmark MS/MS spectral similarity algorithms by self-searching spectra against a high-quality library (e.g., 434,287 NIST20 spectra) under all candidate algorithms and comparing false discovery rate / true-positive rate curves across score thresholds. Used to establish which similarity metric (e.g., entropy similarity vs. dot product) performs best for compound identification.

## When to use

Use when you must compare spectral similarity metrics head-to-head on a spectral library search — e.g., deciding whether entropy similarity beats dot product or 40+ alternatives on NIST20/MassBank-style MS/MS spectra.

## When NOT to use

- Input is raw LC-MS/MS instrument data needing acquisition or peak picking — this skill assumes pre-acquired, precursor-m/z-annotated peak lists
- You need a single universal score cutoff that prevents false positives — the article finds no single similarity threshold prevents false-positive discoveries, so per-threshold FDR/TPR curves, not a fixed cutoff, are the output
- Benchmarking in-silico-only libraries without experimental ground truth, where true/false identification labels cannot be assigned

## Inputs

- High-resolution MS/MS spectra from a licensed spectral library (NIST20, 434,287 spectra; also applicable to MassBank of North America / Massbank.us or GNPS spectra)
- Precursor m/z values for each spectrum (for precursor-ion removal during cleaning)
- Implementations of candidate similarity algorithms (43 total, per Supplementary Note 1)

## Outputs

- Per-algorithm FDR vs. similarity-score threshold curves (e.g., similarity_benchmark_fdr_tpr.csv)
- Per-algorithm true-positive rate curves across score thresholds
- Ranked comparison of all 43 similarity algorithms, with the best performer identified (entropy similarity outperformed the other 42)

## How to apply

Split each spectrum into library and query roles, leaving the test spectrum itself out of the library for self-search, then clean every spectrum with ms_entropy.clean_spectrum: remove empty peaks, remove ions with m/z above precursor m/z - 1.6, centroid peaks within +/- min_ms2_difference_in_da, drop peaks below noise_threshold (e.g., 0.01) * max intensity, and normalize intensities to sum to 1. Compute similarity scores for each query against the library under all algorithms (43 total, including entropy similarity via ms_entropy.calculate_entropy_similarity and dot product), matching fragment ions at 0.05 Da tolerance and removing low-abundance ions below 1% base peak intensity. Aggregate per-algorithm matched scores, classify matches as true or false identifications, and compute FDR and true-positive rate as functions of the similarity-score threshold. Implement the algorithms per Supplementary Note 1 in Python 3.7.9 with benchmarking code in R 3.6.3, and write the per-algorithm FDR/TPR curve data to a comparison table (e.g., similarity_benchmark_fdr_tpr.csv).

## Related tools

- **ms_entropy** (Provides clean_spectrum for spectral pre-processing and calculate_entropy_similarity for the entropy similarity score being benchmarked) — https://github.com/YuanyueLi/MSEntropy
- **MSEntropy** (Current package integrating spectral entropy, entropy similarity, and related functions; recommended over the original SpectralEntropy repo for the most recent version) — https://github.com/YuanyueLi/MSEntropy
- **Python 3.7.9** (Language in which the 43 similarity algorithms were implemented (Supplementary Note 1) for benchmarking) — https://github.com/YuanyueLi/SpectralEntropy
- **R 3.6.3** (Used for the benchmarking analysis code alongside Python)
- **NumPy** (Array representation of spectra (m/z, intensity pairs) passed to ms_entropy functions)
- **SciPy** (scipy.stats entropy function can be used to skip the clean_spectrum step)
- **multiprocessing** (Parallelizes similarity computations across the large spectrum sets)

## Examples

```
import numpy as np, ms_entropy as me; peaks_query = me.clean_spectrum(peaks_query, precursor_mz, min_ms2_difference_in_da=0.05, noise_threshold=0.01); sim = me.calculate_entropy_similarity(peaks_query, peaks_library)
```

## Evaluation signals

- Reproduced result: entropy similarity outperforms all 42 alternative algorithms (including dot product similarity) on the NIST20 benchmark — check the ranking of AUCs/FDR curves
- Spectrum cleaning invariants: empty peaks removed, no ions with m/z above precursor_mz - 1.6 remain, intensities sum to 1 after normalization
- Fragment ion matching uses 0.05 Da tolerance consistently across all 43 algorithms
- Output CSV contains paired FDR/TPR values as functions of score threshold for every algorithm
- Known calibration point: FDR below 10% at entropy similarity score 0.75 on 37,299 experimental natural product spectra

## Limitations

- No single similarity threshold can be defined that prevents false-positive discoveries, even with entropy similarity scores
- The ENTACT study data for overall compound identification schemes is not publicly available, limiting external benchmarking
- Power transformations at w < 1 decrease similarity scoring performance for ion-rich spectra with high entropy values
- The ms_entropy package has since been rewritten with the Flash entropy search algorithm — current versions may differ in speed but preserve accuracy per the README
- Missing version numbers for ms_entropy and numpy in code examples, so exact package versions for reproduction are unstated

## Evidence

- [readme] MSEntropy package is the current code source: [evidence span withheld — anchor status: external_doc]
