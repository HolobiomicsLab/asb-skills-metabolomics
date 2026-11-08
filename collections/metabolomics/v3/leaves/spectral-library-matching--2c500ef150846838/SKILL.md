---
name: spectral-library-matching
description: Use when query EI-MS spectra (e.g., NIST 2017 test spectra) must be identified
  by matching against a million-scale predicted (in-silico) library where exact brute-force
  search is too slow.
when_to_use_negative:
- Query spectra are not electron ionization mass spectra (e.g., LC-MS/MS tandem spectra),
  since the library and Word2vec model were built for EI spectra
- The library is small enough that exact nearest-neighbor search (brute-force WCS)
  is tractable and approximate HNSW search offers no speed benefit
- The target compounds are outside the element/mass scope used to build the library
  (e.g., ionic compounds, mass ≥ 1000 Da, elements beyond H, C, O, N, P, S, Cl, F,
  Br, I, Si), since such molecules were filtered out during library construction
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: FastEI
  role: 'End-to-end spectrum matching pipeline: embeds query spectra with Word2vec
    and searches the HNSW index over the in-silico library'
  repo: https://github.com/Qiong-Yang/FastEI
- name: gensim
  role: Python library used to load the trained Word2vec model and transform spectra
    into low-dimensional embeddings by summing peak embeddings
- name: hnswlib
  role: Approximate nearest-neighbor search over library spectral embeddings via the
    pre-built HNSW index (M = 64, efConstruction = 600, ef = 300); supports incremental
    index updates for library expansion
  repo: https://github.com/nmslib/hnswlib
- name: faiss.IndexFlatL2
  role: Exact baseline (WCS) implemented as Euclidean distance between normalized
    embedding vectors via faiss.IndexFlatL2
- name: RDKit
  role: Reads SMILES strings and computes ECFPs of library molecules that are fed
    into the NEIMS model to generate predicted library spectra
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:01b55ec3149e7c472b2b74740468b6264570322696e85f95eba18b94b3f91629
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
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectral-library-matching

## Summary

Ultra-fast EI mass spectrum matching against a million-scale in-silico library by embedding spectra with a Word2vec peak-embedding model and searching an HNSW approximate nearest-neighbor index (FastEI), achieving 80.4% recall@10 at ~238 spectra/s versus the weighted cosine similarity (WCS) baseline.

## When to use

Use when query EI-MS spectra (e.g., NIST 2017 test spectra) must be identified by matching against a million-scale predicted (in-silico) library where exact brute-force search is too slow.

## When NOT to use

- Query spectra are not electron ionization mass spectra (e.g., LC-MS/MS tandem spectra), since the library and Word2vec model were built for EI spectra
- The library is small enough that exact nearest-neighbor search (brute-force WCS) is tractable and approximate HNSW search offers no speed benefit
- The target compounds are outside the element/mass scope used to build the library (e.g., ionic compounds, mass ≥ 1000 Da, elements beyond H, C, O, N, P, S, Cl, F, Br, I, Si), since such molecules were filtered out during library construction

## Inputs

- Query EI mass spectra as embeddings transformed with references_word2vec.model (e.g., 11,499 NIST 2017 test-set spectra)
- In-silico library database IN_SILICO_LIBRARY.db (~2,146,690 molecules; Zenodo record 7293922)
- Pre-built HNSW index references_index.bin (hnswlib, M = 64, efConstruction = 600, ef = 300)
- Pretrained Word2vec model references_word2vec.model (gensim)
- Optional precursor molecular masses for applying the 5 Da mass filter

## Outputs

- Top-k (top-1, top-10, top-20) candidate molecule IDs per query spectrum with matching scores
- Per-query matching wall time (e.g., ~0.0046 s FastEI vs ~2.6368 s WCS on the expanded library)
- recall@1 / recall@10 metrics per method and filter condition, e.g., benchmark_results.csv

## How to apply

Transform each query spectrum into a low-dimensional embedding by summing per-peak embeddings from the pretrained Word2vec model (references_word2vec.model, trained via gensim on 2,123,695 predicted spectra). Load the pre-built hnswlib index (M = 64, efConstruction = 600, ef = 300) over the ~2,146,690-spectrum in-silico library (IN_SILICO_LIBRARY.db from Zenodo record 7293922) and run approximate nearest-neighbor search, recording the top-10 candidates and per-query wall time. For comparison, run the exact WCS baseline: normalize query and library embeddings and compute Euclidean distances between normalized vectors (equivalent to weighted cosine similarity) via faiss.IndexFlatL2. Optionally apply a 5 Da precursor-mass filter to restrict candidates, which lifts recall@1 from 36.7% to 45.3% and recall@10 from 80.4% to 88.3%. Judge correctness by verifying reproduced recall@1/recall@10 (~36.7/80.4 unfiltered) match the paper within tolerance and that per-query time is ~0.004 s for FastEI versus seconds for brute-force WCS (~2.6 s on the expanded library).

## Related tools

- **FastEI** (End-to-end spectrum matching pipeline: embeds query spectra with Word2vec and searches the HNSW index over the in-silico library) — https://github.com/Qiong-Yang/FastEI
- **gensim** (Python library used to load the trained Word2vec model and transform spectra into low-dimensional embeddings by summing peak embeddings)
- **hnswlib** (Approximate nearest-neighbor search over library spectral embeddings via the pre-built HNSW index (M = 64, efConstruction = 600, ef = 300); supports incremental index updates for library expansion) — https://github.com/nmslib/hnswlib
- **faiss.IndexFlatL2** (Exact baseline (WCS) implemented as Euclidean distance between normalized embedding vectors via faiss.IndexFlatL2)
- **RDKit** (Reads SMILES strings and computes ECFPs of library molecules that are fed into the NEIMS model to generate predicted library spectra)

## Examples

```
git clone https://github.com/Qiong-Yang/FastEI.git && cd FastEI && conda env create -f FastEI.yml && conda activate FastEI && python FastEI.py
```

## Evaluation signals

- recall@1 and recall@10 on the NIST 2017 test set match the reported values (36.7% / 80.4% unfiltered; 45.3% / 88.3% with 5 Da mass filter) within a small tolerance
- Per-query matching time shows the reported ~two-orders-of-magnitude speedup of FastEI (~0.0046 s) over brute-force WCS (~2.6368 s on the expanded 2,253,216-molecule library)
- The EC ablation (embeddings + cosine similarity via faiss.IndexFlatL2) reproduces FastEI-level accuracy (~80.4% recall@10) but only WCS-like speed, confirming accuracy comes from Word2vec embeddings and speed from HNSW
- After incremental library expansion (adding 106,526 f-HMDB/extra-test spectra to reach 2,253,216 molecules), recall@1/recall@10 only slightly decrease to 36.4%/79.9%, and the hnswlib index update succeeds
- All queries return valid top-10 candidate IDs resolvable against IN_SILICO_LIBRARY.db with no missing references

## Limitations

- The distributed FastEI GUI install version supports only Windows 64-bit (tested on Windows 7 and 10)
- FastEI accuracy degrades slightly when the library is expanded beyond NIST-derived compounds: 50% recall@1 is achieved only on molecules beyond the NIST 2017 library, and performance on the 10-molecule extra test set rests on a very small sample
- HNSW is an approximate method, so top-k results may miss the exact nearest neighbor found by brute-force WCS; the 5 Da mass filter is needed to reach 88.3% recall@10
- Ablations show the components are separable: EC gives FastEI-level accuracy only at WCS-like speed, and BH (binning + HNSW) reaches only WCS-level accuracy with partial speed improvement — both components are required for best results

## Evidence

- [discussion] HNSW index parameters from discussion: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [discussion] WCS baseline via faiss from discussion: "They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL2"
- [abstract] Headline accuracy/speed claim from abstract: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [readme] Embedding construction from README: "FastEI is an ultra-fast and accurate spectrum matching method, proposed to improve accuracy by Word2vec-based spectrum embedding and boost the speed using hierarchical navigable small world graph"
- [methods] Environment setup from methods: "conda env create -f FastEI.yml 
      conda activate FastEI"
