---
name: recall-metric-evaluation-spectrum-identification
description: Use when benchmarking a spectrum-to-library identification method on
  measured EI-MS spectra and you must report recall@k accuracy and per-query matching
  time against a similarity-search baseline such as WCS.
when_to_use_negative:
- When the query spectra are tandem MS/MS or LC-MS spectra rather than electron ionization
  (EI) mass spectra matched against the in-silico EI-MS library
- When the question is end-to-end compound identification with mass filtering (e.g.,
  the 5 Da mass filter yielding 88.3% recall@10) rather than a pure top-k matching
  accuracy benchmark
- When ground-truth labels for query spectra are unavailable, since recall@k cannot
  be computed without known correct hits
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FastEI
  role: Approximate nearest-neighbor spectrum matching method (Word2vec embedding
    + HNSW) whose recall@k and speed are evaluated
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Builds and queries the HNSW index used by FastEI for top-k nearest-neighbor
    spectrum search
  repo: https://github.com/nmslib/hnswlib
- name: faiss.IndexFlatL2
  role: Exact Euclidean nearest-neighbor search between normalized vectors implementing
    the weighted cosine similarity (WCS) baseline
- name: gensim
  role: Provides the Word2vec model that transforms predicted spectra into low-dimensional
    embeddings prior to matching
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:e478871d83cad61f1f0fee7bfcbbc4cce797bb4e49c07ec111a33524d37846e9
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# recall-metric-evaluation-spectrum-identification

## Summary

Evaluates compound identification by EI-MS spectrum matching using recall@k metrics (recall@1, recall@10) on measured test spectra queried against a million-scale in-silico library, and compares approximate nearest-neighbor matching (FastEI with Word2vec embeddings + HNSW) against the weighted cosine similarity (WCS) baseline for both accuracy and per-query runtime.

## When to use

Use when benchmarking a spectrum-to-library identification method on measured EI-MS spectra and you must report recall@k accuracy and per-query matching time against a similarity-search baseline such as WCS.

## When NOT to use

- When the query spectra are tandem MS/MS or LC-MS spectra rather than electron ionization (EI) mass spectra matched against the in-silico EI-MS library
- When the question is end-to-end compound identification with mass filtering (e.g., the 5 Da mass filter yielding 88.3% recall@10) rather than a pure top-k matching accuracy benchmark
- When ground-truth labels for query spectra are unavailable, since recall@k cannot be computed without known correct hits

## Inputs

- Test-set query spectra embeddings (low-dimensional vectors obtained by summing peak embeddings with the pretrained Word2vec model via gensim)
- Ground-truth compound identifiers for each query spectrum
- HNSW index over the in-silico library embeddings (e.g., 2,253,216 molecules including f-HMDB and extra test set expansions)
- faiss.IndexFlatL2 index over normalized library vectors for the WCS baseline
- Predicted spectra of library molecules (e.g., IN_SILICO_LIBRARY.db from Zenodo record 7293922)

## Outputs

- recall@1 and recall@10 values for FastEI and WCS on the test set
- Per-query matching times for FastEI versus WCS (seconds per spectrum)
- Benchmark comparison table, e.g., benchmark_results.csv

## How to apply

Match each query spectrum's Word2vec-derived embedding against the library index (hnswlib with M = 64, efConstruction = 600, ef = 300 for FastEI) and, in parallel, run the WCS baseline by computing Euclidean distances between normalized vectors via faiss.IndexFlatL2. Rank the top-k library hits per query and compute recall@1 and recall@10, i.e., the fraction of queries whose ground-truth compound appears in the top-1 or top-10 ranked results. Record FastEI per-query matching time alongside the WCS time to quantify the reported contrast (0.0046 s vs 2.6368 s per spectrum on the expanded 2,253,216-molecule library). Report both metrics for each method (FastEI 36.4/79.9 and WCS 29.7/72.9 for recall@1/recall@10 on the expanded library) and write the comparison to a results table such as benchmark_results.csv. On small held-out sets (e.g., the 10-molecule extra test set beyond the NIST 2017 library), also report recall@20 to reduce sampling noise.

## Related tools

- **FastEI** (Approximate nearest-neighbor spectrum matching method (Word2vec embedding + HNSW) whose recall@k and speed are evaluated) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Builds and queries the HNSW index used by FastEI for top-k nearest-neighbor spectrum search) — https://github.com/nmslib/hnswlib
- **faiss.IndexFlatL2** (Exact Euclidean nearest-neighbor search between normalized vectors implementing the weighted cosine similarity (WCS) baseline)
- **gensim** (Provides the Word2vec model that transforms predicted spectra into low-dimensional embeddings prior to matching)

## Examples

```
labels, _ = index.knn_query(query_vecs, k=10); recall10 = np.mean([gt[i] in labels[i] for i in range(len(gt))]); recall1 = np.mean(labels[:, 0] == gt)
```

## Evaluation signals

- Reproduced FastEI recall@1 ≈ 36.4% and recall@10 ≈ 79.9% on the expanded 2,253,216-molecule library, within a small tolerance
- Reproduced WCS recall@1 ≈ 29.7% and recall@10 ≈ 72.9% on the same expanded library
- FastEI per-query matching time ≈ 0.0046 s versus WCS ≈ 2.6368 s, i.e., roughly a two-orders-of-magnitude speedup
- The extra test set (10 synthetic organic molecules beyond the NIST 2017 library) yields recall@1 = 50%, recall@10 = 70%, recall@20 = 90%
- Every query's ground-truth identifier is present in the library index labels before computing recall@k

## Limitations

- Performance claims on the 10-molecule extra test set rely on a very small sample (recall@20 of 90% corresponds to 9 of 10 compounds)
- HNSW approximate search accuracy depends on ef (set to 300); the ef parameter must be re-set manually after loading a saved index because it is not persisted
- Recall is reported without a mass filter by default; the better 88.3% recall@10 figure requires applying the 5 Da mass filter
- Only approximately 20% of GC-MS compounds can be identified by spectrum matching with current libraries, so low absolute recall@1 does not by itself indicate a pipeline error

## Evidence

- [abstract] FastEI recall and speed vs WCS: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [discussion] HNSW parameters for matching: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [discussion] WCS baseline computation via faiss: "They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL2"
- [discussion] Extra test set recall values: "its recall@1 is 50%, recall@10 is 70%, and recall@20 is 90% by FastEI, respectively"
- [readme] Library matching task definition in NEIMS README: [evidence span withheld — anchor status: external_doc]
