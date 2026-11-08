---
name: approximate-nearest-neighbor-benchmarking
description: Use when you need to quantify the accuracy/speed trade-off of an ANN
  index (e.g., hnswlib HNSW) against exact nearest-neighbor search over a million-scale
  spectral library.
when_to_use_negative:
- The library is small enough that exact nearest-neighbor search is tractable — the
  ANN speedup is only meaningful at million-scale (the article motivates ANN because
  exact search over millions of spectra is time-consuming)
- Query spectra are not electron-ionization mass spectra compatible with the NEIMS-predicted,
  Word2vec-embedded library
- You need exact ground-truth rankings from the ANN method itself — FastEI is approximate
  and must be compared against the WCS baseline, not trusted alone
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FastEI
  role: ANN spectrum matcher combining Word2vec spectral embedding with HNSW search;
    packaged as standalone software for users without programming backgrounds
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: HNSW approximate nearest-neighbor index over library spectral embeddings (M
    = 64, efConstruction = 600, ef = 300)
  repo: https://github.com/nmslib/hnswlib
- name: gensim
  role: Python library used to train the Word2vec model that transforms spectra into
    low-dimensional embeddings
- name: faiss.IndexFlatL2
  role: Exact Euclidean-distance search on normalized vectors, used for the WCS baseline
    comparison
- name: RDKit
  role: Used to read SMILES strings and compute molecular properties (e.g., mass)
    needed for the 5 Da precursor-mass filter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:0c351b9cd62893474daa2451292ad9b30bab395978f849c8bda839346c415177
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# approximate-nearest-neighbor-benchmarking

## Summary

Benchmarks an HNSW-based approximate nearest-neighbor spectral matcher (FastEI) against an exact weighted-cosine-similarity (WCS) baseline on a ~2.15-million-spectrum in-silico EI-MS library, reporting recall@1/recall@10 and per-query wall time with and without a 5 Da precursor-mass filter.

## When to use

Use when you need to quantify the accuracy/speed trade-off of an ANN index (e.g., hnswlib HNSW) against exact nearest-neighbor search over a million-scale spectral library.

## When NOT to use

- The library is small enough that exact nearest-neighbor search is tractable — the ANN speedup is only meaningful at million-scale (the article motivates ANN because exact search over millions of spectra is time-consuming)
- Query spectra are not electron-ionization mass spectra compatible with the NEIMS-predicted, Word2vec-embedded library
- You need exact ground-truth rankings from the ANN method itself — FastEI is approximate and must be compared against the WCS baseline, not trusted alone

## Inputs

- IN_SILICO_LIBRARY.db (2,146,690 predicted EI-MS spectra) from Zenodo record 7293922
- Pre-built hnswlib index file (references_index.bin) with M = 64, efConstruction = 600
- Trained Word2vec model (references_word2vec.model, trained on 2,123,695 predicted spectra via gensim)
- 11,499 query EI-MS spectra from the NIST 2017 test set (spectra/ directory)

## Outputs

- Top-10 candidate rankings per query for FastEI and for the exact WCS baseline
- Per-query wall-time measurements for both methods, with and without the 5 Da mass filter
- benchmark_results.csv with recall@1 and recall@10 for each method under both filtering conditions

## How to apply

Load the pre-built hnswlib index (M = 64, efConstruction = 600, ef = 300) of spectral embeddings for the 2,146,690-molecule in-silico library, embed each of the 11,499 test-set query spectra with the gensim-trained Word2vec model, and run k-NN queries recording top-10 candidates and per-query wall time. Run the exact WCS baseline by brute-force Euclidean distance on normalized embeddings (e.g., faiss.IndexFlatL2) to obtain exact top-10 rankings and wall times. Repeat both searches with a 5 Da precursor-mass filter restricting candidates to library entries within 5 Da of the query molecular mass. Compute recall@1 and recall@10 per method per filtering condition; expect FastEI to reach 36.7% recall@1 / 80.4% recall@10 unfiltered (45.3% / 88.3% with the 5 Da filter) with roughly two-orders-of-magnitude speedup over WCS.

## Related tools

- **FastEI** (ANN spectrum matcher combining Word2vec spectral embedding with HNSW search; packaged as standalone software for users without programming backgrounds) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (HNSW approximate nearest-neighbor index over library spectral embeddings (M = 64, efConstruction = 600, ef = 300)) — https://github.com/nmslib/hnswlib
- **gensim** (Python library used to train the Word2vec model that transforms spectra into low-dimensional embeddings)
- **faiss.IndexFlatL2** (Exact Euclidean-distance search on normalized vectors, used for the WCS baseline comparison)
- **RDKit** (Used to read SMILES strings and compute molecular properties (e.g., mass) needed for the 5 Da precursor-mass filter)

## Examples

```
git clone https://github.com/Qiong-Yang/FastEI.git; cd FastEI; conda env create -f FastEI.yml; conda activate FastEI; cd GUI/ui; python FastEI.py
```

## Evaluation signals

- recall@10 for FastEI without filtering should be approximately 80.4% and recall@1 approximately 36.7% on the 11,499-spectrum test set
- Applying the 5 Da mass filter should raise recall to approximately 45.3% recall@1 and 88.3% recall@10
- Per-query wall time for FastEI should be roughly two orders of magnitude lower than the brute-force WCS baseline
- The exact WCS baseline top-10 rankings (via faiss.IndexFlatL2 on normalized embeddings) should be reproduced identically across reruns (deterministic)
- The in-silico library row count in IN_SILICO_LIBRARY.db should equal 2,146,690 molecules

## Limitations

- FastEI is an approximate method — it can rank the true compound outside top-1 where WCS ranks it top-1, so accuracy must be reported relative to the exact baseline
- The Word2vec hyperparameter table (Supplementary Table 1) is not in the main text, limiting exact re-training of the embedding model
- The packaged FastEI GUI only supports Windows 64-bit (tested on Windows 7/10); Linux users must use the conda development version
- Performance on the 10-molecule extra test set (50% recall@1) relies on a very small sample and molecules beyond the NIST 2017 library

## Evidence

- [abstract] It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS).: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [discussion] The HNSW index of FastEI was created by spectral embeddings of the in-silico library using hnswlib: "The HNSW index of FastEI was created by spectral embeddings of the in-silico library using hnswlib"
- [discussion] They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL2: "They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL2"
- [discussion] using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [methods] Please put [IN_SILICO_LIBRARY.db](https://zenodo.org/record/7293922/files/IN_SILICO_LIBRARY.db): "Please put [IN_SILICO_LIBRARY.db](https://zenodo.org/record/7293922/files/IN_SILICO_LIBRARY.db)"
