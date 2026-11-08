---
name: recall-at-k-benchmark-evaluation
description: Use when comparing spectrum-identification methods on a query test set
  against a large spectral library and you need rank-based accuracy (recall@k) plus
  throughput (spectra/second) metrics.
when_to_use_negative:
- When the library contains only a few thousand spectra — exact nearest-neighbor search
  is feasible and the HNSW approximation overhead is unnecessary
- When evaluating identification of molecules beyond NIST 2017 using the extra test
  set — that task uses recall@1/10/20 on only 10 synthetic compounds and different
  expectations (50%/70%/90%)
- When you need structural similarity scoring rather than spectrum matching — recall@k
  of spectral neighbors does not measure chemical accuracy
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FastEI
  role: 'Full pipeline under test: Word2vec spectral embedding plus HNSW approximate
    nearest-neighbor search over the in-silico library'
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Provides the HNSW approximate nearest-neighbor index queried during benchmarking
    (M = 64, efConstruction = 600, ef = 300)
  repo: https://github.com/nmslib/hnswlib
- name: gensim
  role: Trained Word2vec model used to embed query and library spectra before matching
- name: faiss.IndexFlatL2
  role: Exact Euclidean-distance search on normalized embeddings implementing the
    WCS baseline and EC hybrid
- name: RDKit
  role: Parses SMILES/structures of library molecules and supports the 5 Da mass filter
    via molecular mass
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:0c5ec60ed3d0968c7db57cfb617e35e8f69840d2df222fdff1b4a058caa9fca0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - RDKit
  - faiss.IndexFlatL2
  - gensim
  - hnswlib
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# recall-at-k-benchmark-evaluation

## Summary

Benchmark protocol for evaluating EI mass-spectrum library matching methods (FastEI, WCS, hybrids) by computing recall@1/10/20 and per-query wall time against a million-scale in-silico library, optionally with a 5 Da precursor-mass filter.

## When to use

Use when comparing spectrum-identification methods on a query test set against a large spectral library and you need rank-based accuracy (recall@k) plus throughput (spectra/second) metrics.

## When NOT to use

- When the library contains only a few thousand spectra — exact nearest-neighbor search is feasible and the HNSW approximation overhead is unnecessary
- When evaluating identification of molecules beyond NIST 2017 using the extra test set — that task uses recall@1/10/20 on only 10 synthetic compounds and different expectations (50%/70%/90%)
- When you need structural similarity scoring rather than spectrum matching — recall@k of spectral neighbors does not measure chemical accuracy

## Inputs

- IN_SILICO_LIBRARY.db from Zenodo record 7293922 (~2,146,690 molecules)
- references_index.bin (pre-built HNSW index) and references_word2vec.model from Zenodo record 7293922
- 11,499 test-set measured NIST 2017 query spectra
- FastEI test harness test.py

## Outputs

- benchmark_results.csv with recall@1, recall@10, and per-query wall time for FastEI and WCS, with and without 5 Da mass filter
- hybrid_matching_results.csv with recall@1/10/20 and per-query matching time for EC and BH hybrid configurations
- Throughput metric in spectra/second (FastEI: 238 spectra/second)

## How to apply

Load the ~2.15-million-spectrum in-silico library (IN_SILICO_LIBRARY.db), the pre-built hnswlib HNSW index (M = 64, efConstruction = 600, ef = 300), and the trained Word2vec model (references_index.bin, references_word2vec.model) from Zenodo record 7293922. Embed each of the 11,499 test-set query spectra with the Word2vec model and run the candidate search (FastEI via HNSW, or exact WCS via faiss.IndexFlatL2 on normalized embeddings), recording top-10 candidates and per-query wall time. Repeat the searches with a 5 Da precursor-mass filter restricting candidates to library entries within 5 Da of the query molecular mass. For each method and filtering condition, compute recall@1, recall@10 (and recall@20 for ablations), and per-query matching time, writing all metrics to a results CSV (e.g., benchmark_results.csv or hybrid_matching_results.csv). For component ablations, verify that EC (embeddings + cosine) matches FastEI accuracy at WCS-like speed while BH (binning + HNSW) matches WCS accuracy with partially improved speed.

## Related tools

- **FastEI** (Full pipeline under test: Word2vec spectral embedding plus HNSW approximate nearest-neighbor search over the in-silico library) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Provides the HNSW approximate nearest-neighbor index queried during benchmarking (M = 64, efConstruction = 600, ef = 300)) — https://github.com/nmslib/hnswlib
- **gensim** (Trained Word2vec model used to embed query and library spectra before matching)
- **faiss.IndexFlatL2** (Exact Euclidean-distance search on normalized embeddings implementing the WCS baseline and EC hybrid)
- **RDKit** (Parses SMILES/structures of library molecules and supports the 5 Da mass filter via molecular mass)

## Examples

```
git clone https://github.com/Qiong-Yang/FastEI.git && cd FastEI && conda env create -f FastEI.yml && conda activate FastEI && python test.py  # compute recall@1/10 and per-query time with/without 5 Da filter
```

## Evaluation signals

- FastEI recall@10 should fall at ~80.4% (88.3% with 5 Da mass filter); EC recall@10 should fall within this reported range
- FastEI recall@1 should be ~36.7% (45.3% with 5 Da mass filter) with throughput near 238 spectra/second
- BH configuration should match WCS-level recall while showing a measurable speed improvement over WCS
- FastEI should exhibit roughly a two-orders-of-magnitude speedup over the exact WCS baseline
- All per-configuration metrics should assemble into a single CSV with recall@1/10(/20) and per-query matching time columns

## Limitations

- Recall figures depend on the 2,146,690-molecule in-silico library built from f-NIST and f-ChEMBL NEIMS-predicted spectra; results will differ on other libraries
- HNSW is an approximate search — recall is traded for speed and depends on index parameters (M = 64, efConstruction = 600, ef = 300)
- The extra test set contains only 10 measured spectra, so its recall@k claims rest on a very small sample
- The WCS baseline is referenced for comparison but not defined or cited in the visible text; its exact implementation must be reconstructed

## Evidence

- [abstract] FastEI achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS).: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [intro] FastEI combines Word2vec spectral embedding and HNSW for accuracy and speed.: [evidence span withheld — anchor status: not_found]
- [methods] HNSW index parameters used for FastEI.: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [methods] Euclidean distance between normalized vectors via faiss.: "They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL241"
- [discussion] In-silico library size used for benchmarking.: "the predicted spectra of molecules in f-NIST and f-ChEMBL were used to build the in-silico library, which included 2,146,690 molecules"
- [discussion] FastEI recall on the small extra test set.: "its recall@1 is 50%, recall@10 is 70%, and recall@20 is 90% by FastEI, respectively"
- [readme] Software packaging claim from README.: "FastEI is an ultra-fast and accurate spectrum matching method, proposed to improve accuracy by Word2vec-based spectrum embedding and boost the speed using hierarchical navigable small world graph"
