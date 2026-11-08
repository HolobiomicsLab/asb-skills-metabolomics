---
name: compound-identification-recall-metrics
description: Use when evaluating compound identification accuracy of a spectral library
  search against measured electron ionization mass spectra, especially for molecules
  beyond a reference library, using recall@k metrics.
when_to_use_negative:
- When query spectra match molecules already present in the experimental library (e.g.,
  NIST 2017 with 0.27 million compounds) — that benchmark is reported separately (80.4%
  recall@10) and does not test beyond-library generalization
- When the goal is spectrum matching speed benchmarking against weighted cosine similarity
  (WCS) rather than identification accuracy
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FastEI
  role: Standalone software performing Word2vec spectral embedding and HNSW-based
    spectrum matching against the million-scale in-silico library; test.py computes
    query compound ranks
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Approximate nearest-neighbor search backend (M = 64, efConstruction = 600,
    ef = 300) used for library matching
  repo: https://github.com/nmslib/hnswlib
- name: gensim
  role: Python library used to train the Word2vec model and transform spectra into
    spectral embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:be99a5d79c7f56753161eeacd5d3b08424a85b02349fcf84690ed5322cc6b472
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - gensim
  - hnswlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# compound-identification-recall-metrics

## Summary

Benchmark protocol for quantifying how well FastEI's Word2vec spectral embedding + HNSW approximate nearest-neighbor search identifies compounds absent from an experimental library (NIST 2017), reporting recall@1, recall@10, and recall@20 over query EI-MS spectra.

## When to use

Use when evaluating compound identification accuracy of a spectral library search against measured electron ionization mass spectra, especially for molecules beyond a reference library, using recall@k metrics.

## When NOT to use

- When query spectra match molecules already present in the experimental library (e.g., NIST 2017 with 0.27 million compounds) — that benchmark is reported separately (80.4% recall@10) and does not test beyond-library generalization
- When the goal is spectrum matching speed benchmarking against weighted cosine similarity (WCS) rather than identification accuracy

## Inputs

- 10 measured EI-MS query spectra files (placed in FastEI 'spectra' directory)
- IN_SILICO_LIBRARY.db (million-scale in-silico library, Zenodo record 7293922)
- references_index.bin (precomputed HNSW index)
- references_word2vec.model (pretrained Word2vec spectral embedding model)

## Outputs

- identification_ranks.csv — per-compound rank table (10 rows, one per compound)
- recall@1, recall@10, and recall@20 values over the query set

## How to apply

Place the measured query spectra files into the 'spectra' directory of the FastEI working tree and the IN_SILICO_LIBRARY.db, references_index.bin, and references_word2vec.model files into the 'data' directory so the million-scale in-silico library and its precomputed HNSW index (built with hnswlib, M = 64, efConstruction = 600, ef = 300) are available. Run test.py, which transforms each query spectrum into a Word2vec spectral embedding and performs HNSW approximate nearest-neighbor search against the expanded library. For each of the 10 query spectra, extract the rank of the true compound and record per-compound ranks in a table (identification_ranks.csv). Compute recall@1, recall@10, and recall@20 as the fraction of queries whose true compound appears within the top 1, 10, and 20 matches respectively. Compare the resulting metrics against the reported 50% (5/10), 70% (7/10), and 90% (9/10) values; compounds ranked beyond the evaluated window (e.g., compound 10 with a 12-carbon long-chain substituent ranked >100) explain any recall shortfall.

## Related tools

- **FastEI** (Standalone software performing Word2vec spectral embedding and HNSW-based spectrum matching against the million-scale in-silico library; test.py computes query compound ranks) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Approximate nearest-neighbor search backend (M = 64, efConstruction = 600, ef = 300) used for library matching) — https://github.com/nmslib/hnswlib
- **gensim** (Python library used to train the Word2vec model and transform spectra into spectral embeddings)

## Examples

```
cd FastEI && conda env create -f FastEI.yml && conda activate FastEI && python test.py
```

## Evaluation signals

- Output rank table contains exactly 10 rows, one per compound
- recall@1 = 50% (5/10), recall@10 = 70% (7/10), and recall@20 = 90% (9/10) reproduce the reported values
- Only compound 10 — the molecule carrying a 12-carbon long-chain substituent — is ranked beyond 100
- All other nine compounds rank within the top 20

## Limitations

- Performance claims on the extra test set rest on only 10 measured spectra, a very small sample
- Only supports Windows 64-bit for the packaged GUI install version (tested on Windows 7 and Windows 10)
- FastEI search is approximate (HNSW with ef = 300), so ranks can vary with index/query parameters
- Compound 10 with a 12-carbon long-chain substituent ranked >100, indicating long-chain substituents are a failure mode

## Evidence

- [discussion] On the 10-molecule extra test set, FastEI achieves recall@1 of 50%, recall@10 of 70%, and recall@20 of 90%: "its recall@1 is 50%, recall@10 is 70%, and recall@20 is 90% by FastEI, respectively"
- [discussion] HNSW search parameters: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [discussion] Extra test set size: "It consists of 10 measured spectra. It was used to evaluate the performance of FastEI for molecules beyond the NIST 2017 library"
- [abstract] FastEI described as embedding + HNSW method: "substantially improve accuracy using Word2vec spectral embedding and boost the speed using the hierarchical navigable small-world graph (HNSW)"
- [readme] Development environment setup from README: "conda env create -f FastEI.yml 
      conda activate FastEI"
