---
name: precursor-mass-filtering-spectrum-search
description: Use when matching EI-MS query spectra against a million-scale in-silico
  library and the query's molecular mass is known or estimable, so candidates can
  be restricted to a ±5 Da mass window to improve recall@1/recall@10.
when_to_use_negative:
- The query spectrum has no reliable molecular mass estimate, so a 5 Da window cannot
  be constructed
- The candidate library is small enough that exhaustive exact search (e.g., brute-force
  WCS) is already tractable and the filter's speed benefit is moot
- The query molecules are outside the library's chemical scope (e.g., mass ≥ 1000
  Da, ionic compounds, or elements beyond H, C, O, N, P, S, Cl, F, Br, I, Si), which
  the library filtering steps already excluded
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: FastEI
  role: Runs the Word2vec-embedding + HNSW spectrum search whose candidates are restricted
    by the 5 Da mass filter
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Approximate nearest-neighbor query engine; its knn_query label filter supports
    mass-window candidate restriction
  repo: https://github.com/nmslib/hnswlib
- name: faiss.IndexFlatL2
  role: Exact baseline search on normalized embeddings (WCS) re-run with the same
    5 Da filter for fair comparison
- name: gensim
  role: Provides the trained Word2vec model used to embed query spectra before filtered
    search
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:9b7cbaf06c12370ace3b6960c932bfe3794232b33310371dbd10d5f3031e78f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - faiss.IndexFlatL2
  - gensim
  - hnswlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# precursor-mass-filtering-spectrum-search

## Summary

Restricts spectrum-matching candidates in the FastEI pipeline to in-silico library entries whose molecular mass is within 5 Da of the query's molecular mass, before or after approximate nearest-neighbor search with the HNSW index. This raises identification accuracy (e.g., recall@10 from 80.4% to 88.3%) at little additional cost.

## When to use

Use when matching EI-MS query spectra against a million-scale in-silico library and the query's molecular mass is known or estimable, so candidates can be restricted to a ±5 Da mass window to improve recall@1/recall@10.

## When NOT to use

- The query spectrum has no reliable molecular mass estimate, so a 5 Da window cannot be constructed
- The candidate library is small enough that exhaustive exact search (e.g., brute-force WCS) is already tractable and the filter's speed benefit is moot
- The query molecules are outside the library's chemical scope (e.g., mass ≥ 1000 Da, ionic compounds, or elements beyond H, C, O, N, P, S, Cl, F, Br, I, Si), which the library filtering steps already excluded

## Inputs

- 11,499 test-set EI-MS query spectra with known molecular masses
- IN_SILICO_LIBRARY.db (~2,146,690 predicted spectra) from Zenodo record 7293922
- Pre-built hnswlib index (references_index.bin, M=64, efConstruction=600, ef=300)
- Trained Word2vec spectrum-embedding model (references_word2vec.model)
- Per-library-entry molecular masses for mass-window restriction

## Outputs

- Top-10 candidate lists per query under the 5 Da mass filter
- Filtered recall@1 and recall@10 metrics (expected: 45.3% and 88.3% on the NIST 2017 test set)
- Per-query wall-time measurements with filtering enabled
- benchmark_results.csv rows comparing filtered vs. unfiltered FastEI and WCS

## How to apply

Run the FastEI search (Word2vec spectrum embedding + hnswlib HNSW index with M=64, efConstruction=600, ef=300) once without filtering to record the baseline top-10 candidates and per-query wall time, then re-run the same search with the 5 Da precursor-mass filter applied, restricting candidates to library entries whose molecular mass is within 5 Da of the query molecular mass. The filter can be passed to hnswlib's knn_query via its label-filter argument or applied by pre-partitioning the ~2,146,690-molecule in-silico library by mass. For a fair benchmark, apply the identical filter to the exact WCS baseline (Euclidean distance on normalized embeddings via faiss.IndexFlatL2 / weighted cosine similarity). Finally, recompute recall@1 and recall@10 under both filtered and unfiltered conditions and, if reported, verify the filtered result approaches the published 45.3% recall@1 / 88.3% recall@10.

## Related tools

- **FastEI** (Runs the Word2vec-embedding + HNSW spectrum search whose candidates are restricted by the 5 Da mass filter) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Approximate nearest-neighbor query engine; its knn_query label filter supports mass-window candidate restriction) — https://github.com/nmslib/hnswlib
- **faiss.IndexFlatL2** (Exact baseline search on normalized embeddings (WCS) re-run with the same 5 Da filter for fair comparison)
- **gensim** (Provides the trained Word2vec model used to embed query spectra before filtered search)

## Examples

```
index.knn_query(query_vec, k=10, filter=lambda i: abs(lib_mass[i]-query_mass)<=5.0, num_threads=1)
```

## Evaluation signals

- Filtered recall@1 ≈ 45.3% and recall@10 ≈ 88.3% on the 11,499-spectrum NIST 2017 test set (vs. 36.7% / 80.4% unfiltered)
- Every returned candidate's library molecular mass is within 5 Da of the query molecular mass (window invariant)
- Filtered per-query search time remains within the same order of magnitude as unfiltered FastEI (two-orders-of-magnitude speedup over WCS is preserved)
- benchmark_results.csv contains paired filtered/unfiltered rows for both FastEI and WCS with consistent query counts

## Limitations

- A too-narrow mass window will exclude the true hit if the query's molecular mass is misassigned or the precursor mass is inaccurate
- Filtering in Python multithreaded hnswlib queries is slow; num_threads=1 is recommended for knn_query with a filter
- Reported gains are specific to the ~2.15-million-spectrum in-silico library; performance on the 10-molecule extra test set rests on a very small sample

## Evidence

- [abstract] abstract: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [other] workflow: [evidence span withheld — anchor status: not_found]
- [other] finding: "After applying a 5 Da mass filter to the spectrum matching result, FastEI achieves 45.3% recall@1 and 88.3% recall@10."
- [readme] readme: [evidence span withheld — anchor status: not_found]
