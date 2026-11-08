---
name: spectral-entropy-similarity-scoring
description: Use when comparing MS/MS spectra (query peaks + precursor m/z against
  a spectral library) to rank matches with entropy similarity instead of dot-product/cosine,
  or when benchmarking search speed against LipidIN's EQ module.
when_to_use_negative:
- Searching very large libraries (e.g. LipidIN's 168.6-million-entry theoretical library
  or 10M-spectrum MassBank variants) where LipidIN's EQ module, with its linear-time
  hash-table/bisection data structure (~2.3 µs per query), is faster than Flash Entropy
  Search's squared time complexity
- Feature annotation workflows where similarity-based algorithms introduce redundancy
  — the LipidIN discussion notes similarity algorithms such as MS-DIAL, LipidSearch,
  and MS entropy might cause feature redundancy
- When the query peaks and precursor m/z are not available (only MS1 features), since
  both `precursor_mz` and `peaks` keys are required inputs
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: flash entropy search
  role: Implements the Flash Entropy Search index-and-search algorithm used to compute
    entropy similarity between query spectra and spectral libraries
  repo: https://github.com/YuanyueLi/FlashEntropySearch
- name: Spectral Entropy
  role: Original spectral entropy / entropy similarity source code; superseded by
    the MSEntropy package which re-implements entropy similarity with Flash entropy
    search for speed
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: LipidIN
  role: The EQ querying module being benchmarked against Flash Entropy Search; WMYn-regenerated
    fingerprint spectra are validated with spectral entropy similarity
  repo: https://github.com/LinShuhaiLAB/LipidIN
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-59683-5
    title: 'LipidIN: a comprehensive repository for flash platform-independent annotation
      and reverse lipidomics'
  source_content_hash: sha256:34888f565125ea7cd3e49438dbe4914e55bfaf5c26c429db03547bd3bf8e6556
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YuanyueLi/FlashEntropySearch
  tools:
  - LipidIN
  - Spectral Entropy
  - flash entropy search
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# spectral-entropy-similarity-scoring

## Summary

Scores MS/MS spectral similarity using spectral entropy (entropy similarity) via the Flash Entropy Search algorithm (YuanyueLi/MSEntropy, `ms_entropy` package), used in the LipidIN context both as a baseline similarity method and to validate regenerated lipid fingerprint spectra against reference libraries.

## When to use

Use when comparing MS/MS spectra (query peaks + precursor m/z against a spectral library) to rank matches with entropy similarity instead of dot-product/cosine, or when benchmarking search speed against LipidIN's EQ module.

## When NOT to use

- Searching very large libraries (e.g. LipidIN's 168.6-million-entry theoretical library or 10M-spectrum MassBank variants) where LipidIN's EQ module, with its linear-time hash-table/bisection data structure (~2.3 µs per query), is faster than Flash Entropy Search's squared time complexity
- Feature annotation workflows where similarity-based algorithms introduce redundancy — the LipidIN discussion notes similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy
- When the query peaks and precursor m/z are not available (only MS1 features), since both `precursor_mz` and `peaks` keys are required inputs

## Inputs

- Spectral library as a list of Python dicts, each with required `precursor_mz` and `peaks` keys (optional metadata keys allowed)
- Query MS/MS spectra with `precursor_mz` and m/z-intensity peak arrays (e.g. MassBank-derived libraries, MetabolomicsWorkbench ST001794 query spectra)
- Spectral files in .mgf, .msp, .mzML, or .lbm2 format when using the EntropySearch GUI

## Outputs

- Per-query similarity score arrays keyed by search mode: hybrid_search, identity_search, neutral_loss_search, open_search (numpy float32 arrays, one entry per library spectrum)
- Pairwise entropy similarity scores used to validate regenerated lipid fingerprint spectra against reference libraries
- Benchmark timing tables (e.g. eq_flash_entropy_benchmark.csv with per-query wall-clock time per (method, library-size) pair)

## How to apply

Format library and query spectra as Python dicts with required `precursor_mz` and `peaks` keys (peaks as m/z-intensity arrays, e.g. `np.float32` [[100.0, 1.0], ...]). Build an index from the library with `FlashEntropySearch.build_index(library)`, then call `.search(precursor_mz=..., peaks=...)` to obtain arrays of entropy similarities for four search modes: hybrid_search, identity_search, neutral_loss_search, and open_search. The standalone `ms_entropy` package is installable via pip, and a GUI (EntropySearch) accepts `.mgf`, `.msp`, `.mzML`, and `.lbm2` files for non-programmatic use. In the LipidIN paper, Flash Entropy Search serves as the speed/accuracy comparator for the EQ module: EQ's hash-table/bisection linear-time data structure outperforms Flash Entropy Search's squared time complexity, and entropy/cosine similarity against reference standards is used to validate WMYn-regenerated 5th-level lipid fingerprint libraries. Run single-threaded CPU timing comparisons under identical query sets and library sizes (e.g. MassBank 1M-spectrum library) when benchmarking against EQ.

## Related tools

- **flash entropy search** (Implements the Flash Entropy Search index-and-search algorithm used to compute entropy similarity between query spectra and spectral libraries) — https://github.com/YuanyueLi/FlashEntropySearch
- **Spectral Entropy** (Original spectral entropy / entropy similarity source code; superseded by the MSEntropy package which re-implements entropy similarity with Flash entropy search for speed) — https://github.com/YuanyueLi/SpectralEntropy
- **LipidIN** (The EQ querying module being benchmarked against Flash Entropy Search; WMYn-regenerated fingerprint spectra are validated with spectral entropy similarity) — https://github.com/LinShuhaiLAB/LipidIN

## Examples

```
pip install ms_entropy; from ms_entropy import FlashEntropySearch; es = FlashEntropySearch(); lib = es.build_index(spectral_library); scores = es.search(precursor_mz=150.0, peaks=query_peaks)
```

## Evaluation signals

- Entropy similarity scores fall in the [0, 1] range and the search result dict contains exactly the four keys hybrid_search, identity_search, neutral_loss_search, open_search with one float32 entry per library spectrum
- Index construction requires all library dicts to carry `precursor_mz` and `peaks` keys — a missing key should raise an error or be caught before search
- Benchmark timings are of the expected order of magnitude: ~0.14 s per MS2 query against a 1M-spectrum MassBank library, single-threaded CPU
- When validating WMYn-regenerated lipid fingerprints, spectral-entropy similarity against reference standards should approach the reported ~0.9826 and cosine similarity > 0.96 on NIST SRM 1950 lipids
- Hybrid search should return high similarity (~1.0) for spectra sharing the same peaks as library entries with compatible precursor m/z, as in the README demo where identical spectra score ~0.99999994

## Limitations

- Flash Entropy Search uses squared time complexity, which LipidIN's EQ module (hash tables + bisection, linear time) outperforms at very large library scales
- Similarity-based scoring can cause feature redundancy, as noted for MS-DIAL, LipidSearch, and MS entropy in the article discussion
- Flash entropy search is only available in Python (no R/C/C++/JavaScript implementation of the index-and-search algorithm; the msentropy R package covers entropy similarity calculation only)

## Evidence

- [discussion] data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search: "data structure optimization based on linear time complexity in EQ module promoted by applying hash tables and bisection methods, outperforms squared time complexity in flash entropy search"
- [readme] pip install ms_entropy: [evidence span withheld — anchor status: external_doc]
- [readme] Step 1: Build the index from the library spectra / Step 2: Search the library: "# Step 1: Build the index from the library spectra
entropy_search.build_index(spectral_library)"
- [readme] Note that the precursor_mz and peaks keys are required, the reset of the keys are optional.: [evidence span withheld — anchor status: external_doc]
- [discussion] Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy.: "Similarity algorithms such as MS-DIAL, LipidSearch, and MS entropy might cause feature redundancy."
- [readme] the method for calculating entropy similarity has been rewritten using the Flash entropy search algorithm. This has resulted in speed improvements without compromising accuracy.: [evidence span withheld — anchor status: not_found]
