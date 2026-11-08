---
name: mass-spectral-library-search-benchmarking
description: Use when comparing the query-time scaling of spectral library search
  modules (e.g., linear-time hash-table/bisection methods vs entropy-similarity search)
  across MassBank libraries of growing size.
when_to_use_negative:
- The goal is annotation accuracy or identification quality (e.g., FDR) rather than
  query-time scaling — use an annotation-validation benchmark instead.
- Query spectra or libraries are not in comparable precursor/peak form (precursor_mz
  + peaks arrays required by FlashEntropySearch).
- Only a single fixed library size is available — scaling behavior vs library size
  cannot be assessed.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Source of the EQ module being benchmarked — hash-table and bisection-based
    linear-time spectral querying data structure
  repo: github.com/LinShuhaiLAB/LipidIN
- name: flash entropy search
  role: Baseline comparator — entropy-similarity spectral search timed on the identical
    query set and library sizes
  repo: https://github.com/YuanyueLi/FlashEntropySearch
- name: Spectral Entropy
  role: Similarity metric underlying the Flash Entropy Search baseline (MSEntropy
    package)
  repo: https://github.com/YuanyueLi/SpectralEntropy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:913ed8c15129831b36f3f0b28bcc5523e8870ac90ed816ef9f911117fe54cf8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - Spectral Entropy
  - flash entropy search
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mass-spectral-library-search-benchmarking

## Summary

Benchmarks the wall-clock querying speed of spectral library search algorithms (LipidIN's EQ module vs Flash Entropy Search) as a function of library size on a single-thread, low-memory CPU.

## When to use

Use when comparing the query-time scaling of spectral library search modules (e.g., linear-time hash-table/bisection methods vs entropy-similarity search) across MassBank libraries of growing size.

## When NOT to use

- The goal is annotation accuracy or identification quality (e.g., FDR) rather than query-time scaling — use an annotation-validation benchmark instead.
- Query spectra or libraries are not in comparable precursor/peak form (precursor_mz + peaks arrays required by FlashEntropySearch).
- Only a single fixed library size is available — scaling behavior vs library size cannot be assessed.

## Inputs

- MetabolomicsWorkbench ST001794 query spectra
- MassBank spectral libraries of varying sizes (1M- and 10M-spectrum variants)
- LipidIN EQ module code (hash-table + bisection linear-time querying)
- FlashEntropySearch Python package (ms_entropy)

## Outputs

- eq_flash_entropy_benchmark.csv — per-query wall-clock timings, one row per (method, library-size) pair

## How to apply

Download MetabolomicsWorkbench ST001794 query spectra and prepare MassBank spectral libraries of varying sizes (including 1M- and 10M-spectrum variants) loaded in Python on a machine with at least 16 GB RAM. Load the LipidIN EQ module's hash-table and bisection-based linear-time data structure from the LinShuhaiLAB/LipidIN code, pin execution to a single CPU thread, and time queries of the ST001794 spectra against each library size — the target figure is ~2.3 µs per query against a 10-million-spectrum library (~0.23 ms per billion comparisons). Run the Flash Entropy Search implementation (YuanyueLi/FlashEntropySearch, installed via `pip install ms_entropy`, using `build_index()` then `search()` with precursor_mz and peaks) on the identical query set, library sizes, and single-thread conditions; the reported baseline is 0.14 s per MS2 query against a 1M-spectrum library. Aggregate per-query wall-clock timings into a comparison table with one row per (method, library-size) pair saved as eq_flash_entropy_benchmark.csv; EQ time should remain virtually flat with library size, consistent with ~70 billion spectral comparisons in < 1 s reported for LipidIN.

## Related tools

- **LipidIN** (Source of the EQ module being benchmarked — hash-table and bisection-based linear-time spectral querying data structure) — github.com/LinShuhaiLAB/LipidIN
- **flash entropy search** (Baseline comparator — entropy-similarity spectral search timed on the identical query set and library sizes) — https://github.com/YuanyueLi/FlashEntropySearch
- **Spectral Entropy** (Similarity metric underlying the Flash Entropy Search baseline (MSEntropy package)) — https://github.com/YuanyueLi/SpectralEntropy

## Examples

```
from ms_entropy import FlashEntropySearch; es = FlashEntropySearch(); lib = es.build_index(spectral_library); sim = es.search(precursor_mz=query_spectrum['precursor_mz'], peaks=query_spectrum['peaks'])
```

## Evaluation signals

- EQ per-query time against the 10M-spectrum library falls in the microsecond range (~2.3 µs, i.e., ~0.23 ms per billion comparisons)
- Flash Entropy Search per-MS2-query time against a 1M-spectrum library is on the order of 0.14 s under single-thread conditions
- EQ timing is essentially flat across library sizes (speed independent of library size), while Flash Entropy scales with library size
- Measured throughput is consistent with the reported ~70 billion spectral comparisons in < 1 s for LipidIN
- eq_flash_entropy_benchmark.csv contains one row per (method, library-size) pair with no missing timing cells

## Limitations

- Benchmark reflects single-thread, low-memory CPU conditions; multi-threaded or GPU results are not characterized.
- The Flash Entropy Search algorithm is only available in Python via the ms-entropy package, limiting cross-language timing comparability.
- Timing figures depend on RAM (minimum 16 GB) and hardware; the 2.3 µs / 0.14 s targets are order-of-magnitude reference points, not exact reproductions.

## Evidence

- [results] EQ matching time remains virtually unchanged as library size grows, completing 10,000,000 spectral library queries in only about 2.3 µs: [evidence span withheld — anchor status: not_found]
- [discussion] data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [methods] Download the spectral libraries from Zenodo → https://zenodo.org/records/21421866: "Download the spectral libraries from **[Zenodo → https://zenodo.org/records/21421866](https://zenodo.org/records/21421866)**"
- [readme] Step 1: Build the index from the library spectra; Step 2: Search the library: "# Step 1: Build the index from the library spectra
entropy_search.build_index(spectral_library)"
