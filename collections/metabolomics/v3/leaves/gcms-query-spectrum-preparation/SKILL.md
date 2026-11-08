---
name: gcms-query-spectrum-preparation
description: Use when measured EI-MS query spectra (e.g., compounds absent from the
  NIST 2017 library) must be loaded into FastEI and identified by embedding-based
  nearest-neighbor search against IN_SILICO_LIBRARY.db.
when_to_use_negative:
- The query spectra are not electron ionization GC-MS spectra (FastEI is built and
  benchmarked specifically for EI mass spectra matched against an in-silico EI-MS
  library)
- The target compounds are already covered by an experimental library such as NIST
  2017, where conventional spectrum matching is expected to work
- The goal is de novo structure elucidation rather than library matching — FastEI
  only returns ranked library candidates and cannot propose structures absent from
  the library
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: FastEI
  role: Standalone pipeline that transforms query EI spectra into Word2vec spectral
    embeddings and performs HNSW nearest-neighbor matching against the million-scale
    in-silico library
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Approximate nearest-neighbor search engine for the HNSW index (M = 64, efConstruction
    = 600, ef = 300) used to match query embeddings to library spectra
  repo: https://github.com/nmslib/hnswlib
- name: gensim
  role: Python library underlying the Word2vec model used to embed query spectra before
    search
- name: NEIMS
  role: Pre-trained model (used without re-training) that produced the predicted spectra
    of the million-scale in-silico library the queries are matched against
  repo: https://github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:92aa786cdfe9f416d8e8fbbd9f87aa1674f8d5fd131a6f972402952f0264562b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - NEIMS
  - gensim
  - hnswlib
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gcms-query-spectrum-preparation

## Summary

Prepares measured GC electron-ionization mass spectra so they can be matched by FastEI: query spectra are placed in the FastEI 'spectra' directory, converted to Word2vec spectral embeddings, and searched against the million-scale in-silico library via a prebuilt HNSW index (hnswlib, M = 64, efConstruction = 600, ef = 300).

## When to use

Use when measured EI-MS query spectra (e.g., compounds absent from the NIST 2017 library) must be loaded into FastEI and identified by embedding-based nearest-neighbor search against IN_SILICO_LIBRARY.db.

## When NOT to use

- The query spectra are not electron ionization GC-MS spectra (FastEI is built and benchmarked specifically for EI mass spectra matched against an in-silico EI-MS library)
- The target compounds are already covered by an experimental library such as NIST 2017, where conventional spectrum matching is expected to work
- The goal is de novo structure elucidation rather than library matching — FastEI only returns ranked library candidates and cannot propose structures absent from the library

## Inputs

- Measured GC-EI-MS query spectra files (e.g., the 10 extra-test-set spectra from the organic synthesis laboratory)
- IN_SILICO_LIBRARY.db (million-scale in-silico EI-MS library from Zenodo record 7293922)
- references_index.bin (precomputed HNSW index)
- references_word2vec.model (pretrained Word2vec spectral embedding model)
- FastEI conda environment definition (FastEI.yml)

## Outputs

- Per-compound rank table (identification_ranks.csv, one row per query spectrum)
- recall@1, recall@10, and recall@20 values over the query set
- Nearest-neighbor search results ranking candidate library molecules for each query spectrum

## How to apply

Place the measured query spectra files into the 'spectra' directory of the FastEI working tree, and put the downloaded IN_SILICO_LIBRARY.db, references_index.bin, and references_word2vec.model into the 'data' directory so the million-scale expanded library and its precomputed HNSW index are available. Run test.py from FastEI, which transforms each query spectrum into a Word2vec spectral embedding (embeddings are built by summing peak embeddings, with the model trained on 2,123,695 predicted spectra using gensim) and performs HNSW approximate nearest-neighbor search against the library. Use the exact index parameters from the paper: hnswlib with M = 64, efConstruction = 600, and ef = 300; a 5 Da mass filter can additionally be applied to improve recall. Extract the rank of the true compound for each query spectrum and compute recall@1, recall@10, and recall@20. Save the per-compound rank table as identification_ranks.csv. Note the FastEI development version runs from a conda environment created from FastEI.yml (Python 3.7, with gensim, rdkit, and hnswlib installed via conda-forge/rdkit channels).

## Related tools

- **FastEI** (Standalone pipeline that transforms query EI spectra into Word2vec spectral embeddings and performs HNSW nearest-neighbor matching against the million-scale in-silico library) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Approximate nearest-neighbor search engine for the HNSW index (M = 64, efConstruction = 600, ef = 300) used to match query embeddings to library spectra) — https://github.com/nmslib/hnswlib
- **gensim** (Python library underlying the Word2vec model used to embed query spectra before search)
- **NEIMS** (Pre-trained model (used without re-training) that produced the predicted spectra of the million-scale in-silico library the queries are matched against) — https://github.com/brain-research/deep-molecular-massspec

## Examples

```
git clone https://github.com/Qiong-Yang/FastEI.git; cd FastEI; conda env create -f FastEI.yml; conda activate FastEI; python test.py
```

## Evaluation signals

- Output rank table contains exactly 10 rows (one per query compound)
- recall@1 = 50% (5/10), recall@10 = 70% (7/10), and recall@20 = 90% (9/10) on the 10-compound extra test set
- Only compound 10, which carries a 12-carbon long-chain substituent, is ranked outside the top 100 (>100)
- HNSW search parameters match the reported configuration: M = 64, efConstruction = 600, ef = 300
- Search runs against the 2,146,690-molecule in-silico library built from f-NIST and f-ChEMBL predicted spectra

## Limitations

- The FastEI install/development version only supports Windows 64-bit (tested on Windows 7 and Windows 10); the GUI installer bundles the database, HNSW index, Word2vec model, and example query spectra
- Performance claims on the extra test set rest on only 10 measured spectra, so recall estimates are coarse; compound 10 with a 12-carbon long-chain substituent was ranked >100
- The version or build date of the NEIMS pre-trained weights used to generate the library is not stated (they were used without re-training)
- Detailed experimental conditions for the extra test set measurements are only in Supplementary Note 2, not in the main text

## Evidence

- [discussion] HNSW index parameters: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [discussion] Word2vec embedding construction: "all the predicted spectra were transformed into low-dimensional embeddings by summing the peak embedding to build the HNSW index for library matching"
- [discussion] Word2vec trained with gensim: "the Word2vec model was trained using the Word2vec training set based on gensim39, a Python library"
- [discussion] Extra test set size: "It consists of 10 measured spectra. It was used to evaluate the performance of FastEI for molecules beyond the NIST 2017 library"
- [discussion] Recall targets on extra test set: "its recall@1 is 50%, recall@10 is 70%, and recall@20 is 90% by FastEI, respectively"
- [readme] Bundled resources and platform: [evidence span withheld — anchor status: external_doc]
- [methods] Environment setup: "conda env create -f FastEI.yml"
- [abstract] Mass filter option: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter)"
