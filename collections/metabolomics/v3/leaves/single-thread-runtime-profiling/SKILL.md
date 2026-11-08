---
name: single-thread-runtime-profiling
description: Use when comparing query speed of spectral-search modules across libraries
  of varying size, or when reproducing the LipidIN EQ (~2.3 µs/query, 10M-spectrum
  library) vs Flash Entropy (0.14 s/MS2, 1M library) timing claims under controlled
  single-thread CPU conditions.
when_to_use_negative:
- The goal is annotation accuracy or false discovery rate (e.g., LipidIN's 5.7% FDR
  over 8,923 lipids) rather than runtime — this skill measures speed only.
- The runtime environment is multi-threaded or GPU-accelerated, since the benchmark's
  validity depends on single-thread, low-memory (≥16 GB RAM) conditions.
- The task is small-molecule MS/MS similarity itself (spectral entropy or dot-product)
  rather than timing the search implementations.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LipidIN
  role: Provides the EQ querying module (hash-table and bisection-based linear-time
    data structure) whose per-query wall-clock time is profiled against libraries
    of varying size
  repo: github.com/LinShuhaiLAB/LipidIN
- name: FlashEntropySearch
  role: Baseline method timed on the identical query set, library sizes, and single-thread
    CPU conditions (build_index then search API)
  repo: https://github.com/YuanyueLi/FlashEntropySearch
- name: Spectral Entropy
  role: Underlying entropy-similarity package whose Flash entropy search implementation
    is benchmarked
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:443de4a1621d4d1d07c28805e2fb665ff3d2137e92a7977db310992fe05368ab
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FlashEntropySearch
  - LipidIN
  - Spectral Entropy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-thread-runtime-profiling

## Summary

Benchmarks spectral-library query time as a function of library size by pinning search implementations (LipidIN EQ vs Flash Entropy Search) to a single CPU thread on a low-memory machine and recording per-query wall-clock timings. This isolates algorithmic complexity (linear hash-table/bisection vs squared-time search) from parallel-hardware effects.

## When to use

Use when comparing query speed of spectral-search modules across libraries of varying size, or when reproducing the LipidIN EQ (~2.3 µs/query, 10M-spectrum library) vs Flash Entropy (0.14 s/MS2, 1M library) timing claims under controlled single-thread CPU conditions.

## When NOT to use

- The goal is annotation accuracy or false discovery rate (e.g., LipidIN's 5.7% FDR over 8,923 lipids) rather than runtime — this skill measures speed only.
- The runtime environment is multi-threaded or GPU-accelerated, since the benchmark's validity depends on single-thread, low-memory (≥16 GB RAM) conditions.
- The task is small-molecule MS/MS similarity itself (spectral entropy or dot-product) rather than timing the search implementations.

## Inputs

- MassBank spectral libraries of varying size (including 1M- and 10M-spectrum variants; Zenodo record 21421866)
- MetabolomicsWorkbench ST001794 query spectra
- LipidIN EQ module querying implementation (hash-table and bisection data structure)
- FlashEntropySearch library (YuanyueLi/FlashEntropySearch, pip install ms_entropy)

## Outputs

- eq_flash_entropy_benchmark.csv — per-query wall-clock timings, one row per (method, library-size) pair
- Per-query timing estimates for EQ (~2.3 µs/query at 10M spectra) and Flash Entropy (~0.14 s/MS2 at 1M spectra)

## How to apply

Prepare MassBank spectral libraries at several sizes (including ~1M- and ~10M-spectrum variants) and a fixed query set (e.g., MetabolomicsWorkbench ST001794 spectra), loading everything in Python on a machine with at least 16 GB RAM. Pin execution to a single CPU thread for both methods so the comparison measures the data structure, not parallelism — LipidIN's EQ module uses hash tables and bisection for linear-time complexity, which the paper contrasts with the squared time complexity of Flash Entropy Search. Time EQ queries of the query spectra against each library size, recording per-query wall-clock time, then repeat identically with the FlashEntropySearch implementation (build_index then search per the README API). Aggregate timings into a comparison table with one row per (method, library-size) pair, saved as eq_flash_entropy_benchmark.csv. Validate against reported targets: ~2.3 µs per query on a 10M-spectrum library (~0.23 ms per billion comparisons, ~70 billion comparisons in <1 s) for EQ and ~0.14 s per MS2 query against a 1M-spectrum library for Flash Entropy.

## Related tools

- **LipidIN** (Provides the EQ querying module (hash-table and bisection-based linear-time data structure) whose per-query wall-clock time is profiled against libraries of varying size) — github.com/LinShuhaiLAB/LipidIN
- **FlashEntropySearch** (Baseline method timed on the identical query set, library sizes, and single-thread CPU conditions (build_index then search API)) — https://github.com/YuanyueLi/FlashEntropySearch
- **Spectral Entropy** (Underlying entropy-similarity package whose Flash entropy search implementation is benchmarked) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
from ms_entropy import FlashEntropySearch; es = FlashEntropySearch(); lib = es.build_index(spectral_library); sim = es.search(precursor_mz=q['precursor_mz'], peaks=q['peaks'])
```

## Evaluation signals

- Measured EQ per-query time on the 10M-spectrum library is ~2.3 µs (equivalently ~0.23 ms per billion comparisons), matching the paper's ~70 billion comparisons in <1 s claim.
- Flash Entropy Search per-MS2 query time on the 1M-spectrum library is on the order of 0.14 s.
- EQ timing is approximately flat as library size grows from 1M to 10M spectra, demonstrating library-size independence.
- Both methods were pinned to one CPU thread on the same machine with ≥16 GB RAM, and the identical query set (ST001794) was used for both.
- eq_flash_entropy_benchmark.csv has exactly one row per (method, library-size) pair with wall-clock timings recorded.

## Limitations

- The Flash entropy search algorithm is only available in Python, so the baseline cannot be run in R or other languages (per README).
- Results reflect single-thread, low-memory CPU conditions and may not transfer to multi-threaded or accelerated hardware.
- Validation depends on library-size variants (1M vs 10M spectra) being correctly prepared; malformed library loading would invalidate the size-independence conclusion.

## Evidence

- [discussion] EQ linear vs flash entropy complexity: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [abstract] EQ throughput claim: "The expeditious querying module achieves speeds exceeding one hundred billion queries per second across all mass spectral libraries."
- [readme] FlashEntropySearch API: "# Step 1: Build the index from the library spectra
entropy_search.build_index(spectral_library)"
- [readme] reported throughput figure: "| Query throughput | ~70 billion spectral comparisons in < 1 s |"
