---
name: benchmark-timing-table-generation
description: Use when comparing query speed of spectral library search algorithms
  (e.g., EQ vs flash entropy search) and you need wall-clock timings recorded per
  (method, library-size) pair into a machine-readable benchmark table.
when_to_use_negative:
- When the goal is annotation accuracy or false discovery rate comparison rather than
  query speed — use a full LipidIN annotation workflow instead
- When parallel or multi-thread execution is acceptable, since the reported timings
  (2.3 µs per query, 0.14 s per MS2 query) are defined under single-thread CPU pinning
  and would not be comparable
- When the libraries are not MassBank-derived spectral libraries of varying size —
  the scalability claim (query speed independent of library size) is only demonstrated
  in that setting
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidIN
  role: Source of the EQ module querying implementation (hash-table and bisection-based
    linear-time data structure) that is timed against libraries of varying size
  repo: github.com/LinShuhaiLAB/LipidIN
- name: flash entropy search
  role: Baseline method timed on the identical query set and library sizes; used via
    FlashEntropySearch class with build_index and search steps
  repo: https://github.com/YuanyueLi/FlashEntropySearch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:700bd617eebb696631766f4da6a777c912c9e6f02ed9c615b4c211a0b4113643
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidIN
  - flash entropy search
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# benchmark-timing-table-generation

## Summary

Generates a controlled single-thread timing benchmark comparing LipidIN's EQ querying module against Flash Entropy search across MassBank spectral libraries of varying size, aggregating per-query wall-clock times into a comparison CSV table.

## When to use

Use when comparing query speed of spectral library search algorithms (e.g., EQ vs flash entropy search) and you need wall-clock timings recorded per (method, library-size) pair into a machine-readable benchmark table.

## When NOT to use

- When the goal is annotation accuracy or false discovery rate comparison rather than query speed — use a full LipidIN annotation workflow instead
- When parallel or multi-thread execution is acceptable, since the reported timings (2.3 µs per query, 0.14 s per MS2 query) are defined under single-thread CPU pinning and would not be comparable
- When the libraries are not MassBank-derived spectral libraries of varying size — the scalability claim (query speed independent of library size) is only demonstrated in that setting

## Inputs

- MetabolomicsWorkbench ST001794 query spectra (MS2 spectra)
- MassBank spectral libraries of varying sizes, including 1M-spectrum and 10M-spectrum variants
- LipidIN EQ module querying implementation (hash-table and bisection-based)
- FlashEntropySearch Python package (ms-entropy)

## Outputs

- eq_flash_entropy_benchmark.csv: comparison table with one row per (method, library-size) pair and per-query wall-clock timing
- Measured per-query timings for EQ (target ~2.3 µs per query on 10M spectra) and Flash Entropy (target ~0.14 s per MS2 query on 1M spectra)

## How to apply

Download the MetabolomicsWorkbench ST001794 query spectra and prepare MassBank spectral libraries of varying sizes, including 1M- and 10M-spectrum variants, loading them in Python on a machine with at least 16 GB RAM. Pin execution to a single CPU thread so timing reflects the algorithm rather than parallelism, since the target comparison is EQ's linear-time hash-table and bisection-based data structure against flash entropy search's squared time complexity. Time EQ queries of the ST001794 spectra against each library size, recording per-query wall-clock time (the reported target is 2.3 microseconds per query against a 10-million-spectrum library). Then time the FlashEntropySearch implementation (YuanyueLi/FlashEntropySearch, pip package ms-entropy, using build_index then search) on the identical query set, library sizes, and single-thread conditions, where the reported target is 0.14 s per MS2 query against a 1M-spectrum library. Finally, aggregate measured timings into a table with one row per (method, library-size) pair and save it as eq_flash_entropy_benchmark.csv.

## Related tools

- **LipidIN** (Source of the EQ module querying implementation (hash-table and bisection-based linear-time data structure) that is timed against libraries of varying size) — github.com/LinShuhaiLAB/LipidIN
- **flash entropy search** (Baseline method timed on the identical query set and library sizes; used via FlashEntropySearch class with build_index and search steps) — https://github.com/YuanyueLi/FlashEntropySearch

## Examples

```
pip install ms_entropy; entropy_search = FlashEntropySearch(); library = entropy_search.build_index(spectral_library); entropy_search.search(precursor_mz=q['precursor_mz'], peaks=q['peaks'])
```

## Evaluation signals

- Measured EQ throughput is consistent with the paper's reported performance of ~70 billion spectral comparisons in < 1 s
- Per-query timings fall within the expected order of magnitude of the reported figures: 2.3 µs per EQ query on 10M spectra and 0.14 s per MS2 flash-entropy query on 1M spectra
- EQ matching time remains virtually unchanged as library size grows from small libraries to the 10M-spectrum variant (speed independent of library size)
- Output CSV contains exactly one row per (method, library-size) pair with no missing timing entries
- All runs executed pinned to a single CPU thread on a machine with at least 16 GB RAM

## Limitations

- Flash entropy search scales with squared time complexity relative to EQ's linear-time hash-table/bisection data structure, so timing gaps grow with library size
- Reported timings assume single-thread, low-memory CPU conditions; multi-threaded runs will not reproduce the 2.3 µs / 0.14 s figures
- The Flash entropy search algorithm is only available in Python (via the ms-entropy package), limiting language portability of the benchmark

## Evidence

- [results] EQ matching time remains virtually unchanged as library size grows: [evidence span withheld — anchor status: not_found]
- [discussion] EQ linear-time data structure outperforms flash entropy search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [readme] Reported query throughput target: "| Query throughput | ~70 billion spectral comparisons in < 1 s |"
- [readme] FlashEntropySearch usage pattern: "# Step 1: Build the index from the library spectra
entropy_search.build_index(spectral_library)"
