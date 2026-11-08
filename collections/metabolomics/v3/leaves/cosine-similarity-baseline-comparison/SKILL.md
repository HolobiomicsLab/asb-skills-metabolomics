---
name: cosine-similarity-baseline-comparison
description: Use when benchmarking an approximate nearest-neighbor spectral matcher
  (e.g., FastEI/HNSW) against an exact cosine-similarity reference, or when reporting
  recall@k and per-query time on an in-silico EI-MS library.
when_to_use_negative:
- The task is compound identification itself rather than benchmarking — FastEI alone
  suffices; the WCS baseline is only needed as a reference for accuracy/speedup claims
- The spectral library is small enough that exact nearest-neighbor search is fast
  — the two-order-of-magnitude speedup motivating the comparison does not materialize
- Spectra are not EI mass spectra or cannot be converted to the Word2vec spectral
  embedding space that both FastEI and WCS operate in
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: faiss.IndexFlatL2
  role: 'Exact nearest-neighbor backend for the WCS baseline: computes squared Euclidean
    distances between normalized spectral embedding vectors'
- name: FastEI
  role: 'Method under evaluation: HNSW-based approximate search over Word2vec spectral
    embeddings whose recall and speed are compared to WCS'
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Builds and queries the FastEI HNSW index (M = 64, efConstruction = 600, ef
    = 300) that is benchmarked against WCS
  repo: https://github.com/nmslib/hnswlib
- name: gensim
  role: Loads the pretrained Word2vec model used to convert predicted spectra into
    low-dimensional embeddings for both FastEI and WCS matching
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:90ffa38a88e006dc800c1ff88ef99ad302bda5b16752f705f10b7569432537a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - faiss.IndexFlatL2
  - gensim
  - hnswlib
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cosine-similarity-baseline-comparison

## Summary

Establishes a weighted cosine similarity (WCS) exact-search baseline for EI mass spectrum matching against the FastEI HNSW index, using Euclidean distances between normalized spectral embedding vectors computed with faiss.IndexFlatL2. It quantifies FastEI's accuracy and speedup (two orders of magnitude) on million-scale in-silico libraries.

## When to use

Use when benchmarking an approximate nearest-neighbor spectral matcher (e.g., FastEI/HNSW) against an exact cosine-similarity reference, or when reporting recall@k and per-query time on an in-silico EI-MS library.

## When NOT to use

- The task is compound identification itself rather than benchmarking — FastEI alone suffices; the WCS baseline is only needed as a reference for accuracy/speedup claims
- The spectral library is small enough that exact nearest-neighbor search is fast — the two-order-of-magnitude speedup motivating the comparison does not materialize
- Spectra are not EI mass spectra or cannot be converted to the Word2vec spectral embedding space that both FastEI and WCS operate in

## Inputs

- Query EI mass spectra (test set from NIST 2017; e.g., over ten thousand measured spectra) with ground-truth compound labels
- Library spectral embeddings produced by summing peak embeddings from the pretrained Word2vec model (references_word2vec.model via gensim)
- Word2vec-embedded query spectra, normalized to unit length
- Optional expanded library additions: 106,526 predicted spectra from f-HMDB and the extra test set merged with IN_SILICO_LIBRARY.db (Zenodo record 7293922)

## Outputs

- recall@1 and recall@10 for FastEI and the WCS baseline (expected: 36.4%/79.9% vs 29.7%/72.9% on the expanded library)
- Per-query matching times (expected: 0.0046 s FastEI vs 2.6368 s WCS)
- benchmark_results.csv with the recall and runtime comparison for the two methods

## How to apply

Represent each library and query spectrum as a Word2vec spectral embedding obtained by summing peak embeddings with the pretrained Word2vec model (references_word2vec.model, loaded via gensim). Normalize all embedding vectors to unit length so that squared Euclidean distance is monotonic in cosine similarity, then compute exact nearest neighbors for every query with faiss.IndexFlatL2. Run the same query spectra through FastEI's HNSW index (hnswlib, M = 64, efConstruction = 600, ef = 300) in parallel. Compute recall@1 and recall@10 for both methods and record per-query matching time; on the expanded 2,253,216-molecule library the expected contrast is recall@1 of 36.4% (FastEI) vs 29.7% (WCS), recall@10 of 79.9% vs 72.9%, and 0.0046 s vs 2.6368 s per query. Write the recall and runtime comparison to benchmark_results.csv so accuracy and speed can be judged jointly rather than in isolation.

## Related tools

- **faiss.IndexFlatL2** (Exact nearest-neighbor backend for the WCS baseline: computes squared Euclidean distances between normalized spectral embedding vectors)
- **FastEI** (Method under evaluation: HNSW-based approximate search over Word2vec spectral embeddings whose recall and speed are compared to WCS) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Builds and queries the FastEI HNSW index (M = 64, efConstruction = 600, ef = 300) that is benchmarked against WCS) — https://github.com/nmslib/hnswlib
- **gensim** (Loads the pretrained Word2vec model used to convert predicted spectra into low-dimensional embeddings for both FastEI and WCS matching)

## Examples

```
import faiss; index = faiss.IndexFlatL2(dim); index.add(normalized_library_embeddings); D, I = index.search(normalized_query_embeddings, 10)
```

## Evaluation signals

- Recall@1 and recall@10 for WCS reproduce the reported 29.7% / 72.9% (and FastEI 36.4% / 79.9%) on the expanded 2,253,216-molecule library within a small tolerance
- Per-query matching time reproduces the reported contrast of ~0.0046 s for FastEI vs ~2.6368 s for WCS, i.e., roughly a two-order-of-magnitude speedup
- Library size is verified as 2,146,690 (in-silico) or 2,253,216 (expanded) molecules before matching
- All query and library embedding vectors are normalized before faiss.IndexFlatL2 search, otherwise WCS distances are not cosine-comparable
- benchmark_results.csv exists and contains recall and runtime columns for both methods

## Limitations

- The WCS baseline is exact and scales poorly: 2.6368 s per query against ~2.25 million spectra, making large-scale sweeps impractical
- FastEI recall@1 drops to 50% for molecules beyond the NIST 2017 library, so the comparison's accuracy figures are library-dependent
- WCS is referenced for comparison but not formally defined or cited in the visible text, and the Word2vec hyperparameter table (Supplementary Table 1) is not included
- Recall figures for the extra test set rest on only 10 measured spectra, so accuracy estimates there are statistically fragile

## Evidence

- [discussion] They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL241: "They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL241"
- [discussion] using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [abstract] It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS).: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [discussion] all the predicted spectra were transformed into low-dimensional embeddings by summing the peak embedding to build the HNSW index for library matching: "all the predicted spectra were transformed into low-dimensional embeddings by summing the peak embedding to build the HNSW index for library matching"
- [discussion] the Word2vec model was trained using the Word2vec training set based on gensim39, a Python library: "the Word2vec model was trained using the Word2vec training set based on gensim39, a Python library"
