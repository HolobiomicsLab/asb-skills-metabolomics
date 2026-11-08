---
name: word2vec-spectral-embedding
description: Use when predicted or measured EI mass spectra must be converted into
  fixed-length dense vectors for approximate nearest-neighbor library matching, e.g.
  before building or querying an HNSW index over a million-scale in-silico EI-MS library.
when_to_use_negative:
- When the task is raw experimental-spectrum comparison against an experimental library
  (e.g., NIST 2017 alone) rather than fast matching of a million-scale in-silico library
  — a direct similarity method may suffice.
- When spectra were not generated in the same peak-token format used to train the
  Word2vec model; mismatched tokenization/embedding spaces will corrupt nearest-neighbor
  results.
- When the downstream index is an exact brute-force vector search rather than HNSW-based
  FastEI matching and embedding training cost is not justified.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: gensim
  role: Python library used to train the Word2vec model on the spectral corpus and
    to load references_word2vec.model for transforming spectra into embeddings
  repo: https://github.com/Qiong-Yang/FastEI
- name: FastEI
  role: Pipeline that consumes the Word2vec spectral embeddings and performs HNSW
    approximate nearest-neighbor spectrum matching against the in-silico library
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Builds the HNSW index (M = 64, efConstruction = 600, ef = 300) over the Word2vec
    spectral embeddings of the library
  repo: https://github.com/nmslib/hnswlib
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:b1e3e4962895aad75a598811964561837784af4de4d644436b37bd39db60c255
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - faiss.IndexFlatL2
  - gensim
  - hnswlib
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# word2vec-spectral-embedding

## Summary

Transforms electron ionization mass spectra into low-dimensional spectral embeddings by treating each spectrum as a 'sentence' of peak tokens and summing peak embeddings from a gensim Word2vec model (CBOW, dim 500, window 1000), enabling more accurate compound identification via FastEI's HNSW matching.

## When to use

Use when predicted or measured EI mass spectra must be converted into fixed-length dense vectors for approximate nearest-neighbor library matching, e.g. before building or querying an HNSW index over a million-scale in-silico EI-MS library.

## When NOT to use

- When the task is raw experimental-spectrum comparison against an experimental library (e.g., NIST 2017 alone) rather than fast matching of a million-scale in-silico library — a direct similarity method may suffice.
- When spectra were not generated in the same peak-token format used to train the Word2vec model; mismatched tokenization/embedding spaces will corrupt nearest-neighbor results.
- When the downstream index is an exact brute-force vector search rather than HNSW-based FastEI matching and embedding training cost is not justified.

## Inputs

- Predicted-spectra corpus formatted as peak-token 'sentences' (2,123,695 spectra from f-NIST training set and f-ChEMBL)
- Pretrained Word2vec model file (references_word2vec.model) for embedding transforms via gensim
- Predicted spectra of f-NIST, f-ChEMBL, f-HMDB, and the extra test set (produced by the NEIMS model from ECFPs)

## Outputs

- Trained gensim Word2vec model serialized as references_word2vec.model (500-dimensional peak embeddings)
- Low-dimensional spectral embeddings for all library spectra, used to build the HNSW index (hnswlib, M = 64, efConstruction = 600, ef = 300)
- Query-spectrum embeddings for FastEI matching against the in-silico or expanded library (2,253,216 molecules)

## How to apply

Format each spectrum from the predicted-spectra corpus (e.g., the 2,123,695 spectra from f-NIST and f-ChEMBL in the in-silico library) as a 'sentence' of peak tokens for gensim Word2vec. Train with the paper's hyperparameters: CBOW architecture, window size 1000, embedding dimension 500, negative sampling 5, initial learning rate 0.025 with linear decay 0.00025, for 60 epochs. Once trained (or when loading the pretrained references_word2vec.model), transform every library spectrum into an embedding by summing its peak embeddings; the rationale is that this learned spectral representation substantially improves matching accuracy over raw spectral comparison. Hyperparameters were optimized on a validation set using FastEI's matched results, so if retraining, tune against recall@k of the downstream HNSW match. Serialize the model to the data directory so FastEI can load it for embedding query spectra identically at match time.

## Related tools

- **gensim** (Python library used to train the Word2vec model on the spectral corpus and to load references_word2vec.model for transforming spectra into embeddings) — https://github.com/Qiong-Yang/FastEI
- **FastEI** (Pipeline that consumes the Word2vec spectral embeddings and performs HNSW approximate nearest-neighbor spectrum matching against the in-silico library) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Builds the HNSW index (M = 64, efConstruction = 600, ef = 300) over the Word2vec spectral embeddings of the library) — https://github.com/nmslib/hnswlib

## Examples

```
conda env create -f FastEI.yml; conda activate FastEI; python FastEI.py
```

## Evaluation signals

- references_word2vec.model exists and loads as a valid gensim Word2vec model with 500-dimensional vectors
- Hyperparameter configuration matches the paper's settings: CBOW, window 1000, dim 500, negative 5, lr 0.025, decay 0.00025, 60 epochs
- Downstream FastEI matching reaches recall@1 ≈ 36.4% and recall@10 ≈ 79.9% on the expanded 2,253,216-molecule library (small tolerance)
- On the 10-compound extra test set, recall@1 = 50%, recall@10 = 70%, recall@20 = 90%, with only compound 10 ranked beyond top 100
- Matching speed contrast vs WCS baseline reproduced: ~0.0046 s per spectrum (FastEI) vs ~2.6368 s (WCS)

## Limitations

- All Word2vec hyperparameters are listed only in Supplementary Table 1, which is not included in the main text — full retraining reproducibility depends on the supplementary material.
- The pretrained NEIMS weights used to generate the predicted-spectra training corpus were downloaded without re-training and their version/build date is not stated.
- FastEI's released GUI/installer supports Windows 64-bit only (tested on Windows 7 and 10); the embedding pipeline must run within the FastEI conda environment (Python 3.7).

## Evidence

- [results] FastEI converts each spectrum into a spectral embedding using Word2vec, and these embeddings are indexed with HNSW, which retrieves similar candidate spectra from the library.: [evidence span withheld — anchor status: not_found]
- [discussion] Word2vec trained with gensim and its training corpus: "the Word2vec model was trained using the Word2vec training set based on gensim"
- [discussion] HNSW index parameters via hnswlib: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [discussion] Embeddings built by summing peak embeddings of predicted spectra: "all the predicted spectra were transformed into low-dimensional embeddings by summing the peak embedding to build the HNSW index for library matching"
- [discussion] Word2vec corpus of 2,123,695 predicted spectra: "It consists of 2,123,695 predicted spectra"
- [abstract] FastEI description: accuracy via Word2vec spectral embedding, speed via HNSW: "substantially improve accuracy using Word2vec spectral embedding and boost the speed using the hierarchical navigable small-world graph (HNSW)"
