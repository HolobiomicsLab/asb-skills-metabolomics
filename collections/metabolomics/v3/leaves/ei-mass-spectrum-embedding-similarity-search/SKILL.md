---
name: ei-mass-spectrum-embedding-similarity-search
description: Use when query EI mass spectra must be identified against a million-scale
  in-silico library where exact nearest-neighbor matching is too slow, and recall@1/recall@10
  accuracy plus per-query wall time must be benchmarked.
when_to_use_negative:
- The compound of interest is already covered by an experimental library such as NIST
  2017, where standard library matching suffices and only ~20% of GC-MS compounds
  were historically identifiable
- Query spectra are not electron ionization spectra (e.g., LC-MS/MS tandem spectra),
  since the in-silico library and embeddings are EI-specific
- Very small candidate sets where exhaustive weighted cosine similarity is cheap enough
  that HNSW approximate search provides no speed benefit
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: FastEI
  role: Standalone software implementing Word2vec spectral embedding plus HNSW-based
    approximate nearest-neighbor search against the in-silico library
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Builds and queries the HNSW graph index (M = 64, efConstruction = 600, ef
    = 300) over library spectral embeddings
  repo: https://github.com/nmslib/hnswlib
- name: gensim
  role: Python library used to train the Word2vec model that converts spectra into
    low-dimensional embeddings
- name: faiss.IndexFlatL2
  role: Computes Euclidean distance between normalized vectors for the exact WCS baseline
- name: NEIMS
  role: Pre-trained model (used without re-training) that predicts EI mass spectra
    from ECFP fingerprints to build the in-silico library
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: RDKit (v2022.03.3)
  role: Reads SMILES strings and generates ECFP fingerprints fed into the NEIMS model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:a74f6d9cd985c45d1ed7c2000c508e9dfe07c531d0e588a6c95105161d399a41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - NEIMS
  - RDKit (v2022.03.3)
  - faiss.IndexFlatL2
  - gensim
  - hnswlib
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# ei-mass-spectrum-embedding-similarity-search

## Summary

FastEI-style EI mass spectrum matching that embeds spectra with a Word2vec model and searches a million-scale in-silico library (~2,146,690 spectra) via an hnswlib HNSW index, achieving 80.4% recall@10 (88.3% with a 5 Da mass filter) with ~100x speedup over exact weighted cosine similarity (WCS).

## When to use

Use when query EI mass spectra must be identified against a million-scale in-silico library where exact nearest-neighbor matching is too slow, and recall@1/recall@10 accuracy plus per-query wall time must be benchmarked.

## When NOT to use

- The compound of interest is already covered by an experimental library such as NIST 2017, where standard library matching suffices and only ~20% of GC-MS compounds were historically identifiable
- Query spectra are not electron ionization spectra (e.g., LC-MS/MS tandem spectra), since the in-silico library and embeddings are EI-specific
- Very small candidate sets where exhaustive weighted cosine similarity is cheap enough that HNSW approximate search provides no speed benefit

## Inputs

- IN_SILICO_LIBRARY.db (in-silico EI-MS library database, ~2,146,690 spectra)
- references_index.bin (pre-built hnswlib HNSW index, M=64, efConstruction=600, ef=300)
- references_word2vec.model (trained gensim Word2vec model, trained on 2,123,695 predicted spectra)
- Query EI mass spectra (e.g., 11,499 measured NIST 2017 test-set spectra)

## Outputs

- Top-10 candidate compound matches per query spectrum
- Per-query wall-time measurements for FastEI and the WCS baseline
- recall@1 and recall@10 metrics, with and without the 5 Da mass filter (benchmark_results.csv)

## How to apply

Download IN_SILICO_LIBRARY.db, references_index.bin, and references_word2vec.model from Zenodo record 7293922 into the FastEI data/ directory and place the query spectra (e.g., the 11,499-spectrum NIST 2017 test set) in the spectra/ directory. Load the pre-built hnswlib HNSW index (M = 64, efConstruction = 600, ef = 300) and embed each query spectrum by summing Word2vec peak embeddings from the trained gensim model, then run approximate nearest-neighbor search over the ~2.15-million-spectrum library, recording top-10 candidates and per-query wall time. Optionally rerun the search with a 5 Da precursor-mass filter restricting candidates to library entries within 5 Da of the query molecular mass to boost recall@1 from 36.7% to 45.3% and recall@10 from 80.4% to 88.3%. For benchmarking, compare against an exact WCS baseline computed as Euclidean distance between normalized embeddings (faiss.IndexFlatL2), which is two orders of magnitude slower. Judge success by reproducing recall@1 ≈ 36.7% (45.3% filtered) and recall@10 ≈ 80.4% (88.3% filtered) on the 11,499-spectrum test set.

## Related tools

- **FastEI** (Standalone software implementing Word2vec spectral embedding plus HNSW-based approximate nearest-neighbor search against the in-silico library) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Builds and queries the HNSW graph index (M = 64, efConstruction = 600, ef = 300) over library spectral embeddings) — https://github.com/nmslib/hnswlib
- **gensim** (Python library used to train the Word2vec model that converts spectra into low-dimensional embeddings)
- **faiss.IndexFlatL2** (Computes Euclidean distance between normalized vectors for the exact WCS baseline)
- **NEIMS** (Pre-trained model (used without re-training) that predicts EI mass spectra from ECFP fingerprints to build the in-silico library) — https://github.com/brain-research/deep-molecular-massspec
- **RDKit (v2022.03.3)** (Reads SMILES strings and generates ECFP fingerprints fed into the NEIMS model)

## Examples

```
conda env create -f FastEI.yml; conda activate FastEI; cd GUI/ui; python FastEI.py
```

## Evaluation signals

- recall@1 on the 11,499-spectrum test set should be ~36.7% (45.3% after applying the 5 Da mass filter)
- recall@10 on the test set should be ~80.4% (88.3% after the 5 Da mass filter)
- FastEI per-query wall time should be roughly two orders of magnitude faster than the exact WCS baseline
- The loaded HNSW index must cover ~2,146,690 library molecules and queries must use ef = 300 search parameters
- Benchmark accuracy and timing metrics should be assembled into benchmark_results.csv

## Limitations

- FastEI packaged GUI only supports Windows 64-bit (tested on Windows 7 and 10); the development version requires an Anaconda environment from FastEI.yml
- Performance claims on the 10-molecule extra test set (recall@1 = 50%, recall@10 = 70%, recall@20 = 90%) rely on a very small sample
- The Word2vec hyperparameter table (Supplementary Table 1) is not present in the main text, complicating re-training from scratch
- The WCS baseline comparison method is referenced but not formally defined or cited in the visible text
- The in-silico library is limited to molecules with mass < 1000 Da, containing only 11 common elements (H, C, O, N, P, S, Cl, F, Br, I, Si), non-ionic, with LogP between -12 and 24

## Evidence

- [abstract] It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS).: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [discussion] using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [discussion] the Word2vec model was trained using the Word2vec training set based on gensim39, a Python library: "the Word2vec model was trained using the Word2vec training set based on gensim39, a Python library"
- [methods] git clone https://github.com/Qiong-Yang/FastEI.git: "git clone https://github.com/Qiong-Yang/FastEI.git"
- [discussion] The HNSW index of FastEI was created by spectral embeddings of the in-silico library using hnswlib: "The HNSW index of FastEI was created by spectral embeddings of the in-silico library using hnswlib"
