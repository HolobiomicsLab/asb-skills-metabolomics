---
name: lipidomics-query-throughput-reproduction
description: Use when reproducing or verifying claims that spectral-library query
  time is independent of library size, or when benchmarking MS/MS search engines (LipidIN
  EQ vs Flash Entropy Search) for per-query latency.
when_to_use_negative:
- The goal is annotation accuracy, lipid coverage, or false-discovery rate of LipidIN
  rather than raw query throughput/speed benchmarking.
- Queries will be run multi-threaded or against libraries smaller than the reported
  size variants, so results are not comparable to the single-thread 2.3 µs / 0.14
  s figures.
- The task requires Flash Entropy Search's other search modes (hybrid, neutral loss,
  open search) rather than timing the search itself.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: LipidIN
  role: Source of the EQ module querying implementation (hash-table and bisection-based
    linear-time data structure) whose per-query timing is benchmarked
  repo: github.com/LinShuhaiLAB/LipidIN
- name: flash entropy search
  role: Baseline comparator; spectral-entropy-similarity search timed on identical
    query set, library sizes, and single-thread CPU conditions
  repo: https://github.com/YuanyueLi/MSEntropy
- name: Spectral Entropy
  role: Similarity metric underlying the Flash Entropy Search baseline
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:239d6876523ae1e59fcc18c651730e077773f10fb9c5fa5824bc9024657c957c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - Spectral Entropy
  - flash entropy search
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lipidomics-query-throughput-reproduction

## Summary

Benchmark the LipidIN EQ module's library-size-independent query speed (hash-table/bisection linear-time data structure) against Flash Entropy Search by timing per-query wall-clock latency of MassBank library searches on a single-thread, low-memory CPU.

## When to use

Use when reproducing or verifying claims that spectral-library query time is independent of library size, or when benchmarking MS/MS search engines (LipidIN EQ vs Flash Entropy Search) for per-query latency.

## When NOT to use

- The goal is annotation accuracy, lipid coverage, or false-discovery rate of LipidIN rather than raw query throughput/speed benchmarking.
- Queries will be run multi-threaded or against libraries smaller than the reported size variants, so results are not comparable to the single-thread 2.3 µs / 0.14 s figures.
- The task requires Flash Entropy Search's other search modes (hybrid, neutral loss, open search) rather than timing the search itself.

## Inputs

- MetabolomicsWorkbench ST001794 query spectra
- MassBank spectral libraries of varying size (1M- and 10M-spectrum variants)
- LipidIN EQ module querying implementation (hash-table + bisection linear-time data structure)
- Flash Entropy Search Python implementation (ms_entropy package, FlashEntropySearch class)

## Outputs

- eq_flash_entropy_benchmark.csv — comparison table with one row per (method, library-size) pair of measured per-query wall-clock timings

## How to apply

Download the MetabolomicsWorkbench ST001794 query spectra and prepare MassBank spectral libraries of varying sizes (1M- and 10M-spectrum variants), loading them in Python on a machine with at least 16 GB RAM. Load the EQ module implementation from the LinShuhaiLAB/LipidIN code — a hash-table and bisection-based linear-time data structure — and pin execution to a single CPU thread. Time EQ queries of the ST001794 spectra against each library size, recording per-query wall-clock time (target: 2.3 µs per query against a 10-million-spectrum library, ~0.23 ms per billion comparisons). Time the Flash Entropy Search implementation (YuanyueLi/FlashEntropySearch, installed via `pip install ms_entropy`, using `FlashEntropySearch.build_index()` then `.search()`) on the identical query set, library sizes, and single-thread conditions (target: 0.14 s per MS2 query against a 1M-spectrum library). Aggregate timings into one row per (method, library-size) pair and save as eq_flash_entropy_benchmark.csv.

## Related tools

- **LipidIN** (Source of the EQ module querying implementation (hash-table and bisection-based linear-time data structure) whose per-query timing is benchmarked) — github.com/LinShuhaiLAB/LipidIN
- **flash entropy search** (Baseline comparator; spectral-entropy-similarity search timed on identical query set, library sizes, and single-thread CPU conditions) — https://github.com/YuanyueLi/MSEntropy
- **Spectral Entropy** (Similarity metric underlying the Flash Entropy Search baseline) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
pip install ms_entropy; entropy_search = FlashEntropySearch(); spectral_library = entropy_search.build_index(spectral_library); entropy_similarity = entropy_search.search(precursor_mz=query_spectrum['precursor_mz'], peaks=query_spectrum['peaks'])
```

## Evaluation signals

- Measured EQ per-query time stays virtually unchanged as library size grows (10,000,000 queries in ~2.3 µs against the 10M-spectrum library).
- EQ throughput is consistent with the paper's reported ~70 billion spectral comparisons in < 1 s.
- Flash Entropy Search timing falls in the expected order of magnitude (~0.14 s per MS2 query against a 1M-spectrum library).
- eq_flash_entropy_benchmark.csv contains one row per (method, library-size) pair with wall-clock timings for both methods under identical single-thread conditions.

## Limitations

- Flash Entropy Search exhibits squared time complexity versus EQ's linear time, so relative speed depends strongly on library size.
- Benchmark must be pinned to a single CPU thread; multi-threaded runs are not comparable to the reported 2.3 µs / 0.14 s figures.
- Machine must have at least 16 GB RAM to load the larger (10M-spectrum) library variants.

## Evidence

- [results] EQ matching time remains virtually unchanged as library size grows, completing 10,000,000 spectral library queries in only about 2.3 µs: [evidence span withheld — anchor status: not_found]
- [discussion] linear-time EQ data structure outperforms Flash Entropy Search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [readme] Query throughput reported in README figures: "Query throughput | ~70 billion spectral comparisons in < 1 s"
- [readme] Flash Entropy Search usage from MSEntropy README: "# Step 1: Build the index from the library spectra
entropy_search.build_index(spectral_library)"
