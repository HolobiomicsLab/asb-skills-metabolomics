---
name: ei-mass-spectrum-library-matching
description: Use when measured GC-EI-MS spectra must be matched against a large in-silico
  spectral library, especially for compounds beyond NIST 2017, and exact nearest-neighbor
  search over millions of spectra is too slow.
when_to_use_negative:
- Input spectra were acquired on ionization modes other than electron ionization (EI)
  GC-MS, since FastEI's library and embedding model are trained on EI spectra
- The compound spectrum is already present in the NIST 2017 experimental library and
  a simple spectrum matching search suffices — FastEI's advantage targets molecules
  beyond the 0.27 million-compound NIST 2017 library
- You need an exact brute-force nearest-neighbor result rather than HNSW approximate
  search (approximate search may occasionally misrank, as with compound 10 ranked
  outside the top 100)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: FastEI
  role: Standalone pipeline that transforms query EI spectra into Word2vec embeddings
    and performs HNSW library matching against the million-scale in-silico library
  repo: https://github.com/Qiong-Yang/FastEI
- name: hnswlib
  role: Approximate nearest-neighbor search engine over library spectral embeddings
    (M = 64, efConstruction = 600, ef = 300)
  repo: https://github.com/nmslib/hnswlib
- name: gensim
  role: Python library used to train the Word2vec model that embeds spectra as low-dimensional
    vectors for matching
- name: NEIMS
  role: Pre-trained model used (without re-training) to predict the in-silico mass
    spectra that populate the million-scale library
  repo: https://github.com/brain-research/deep-molecular-massspec
- name: RDKit
  role: Used (v2022.03.3) to read SMILES strings of molecules and compute ECFPs fed
    into NEIMS for library spectrum prediction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:f3ee3518950175f8e0b84883ddbd41b78bab5917970a04c486f9536c0e24a103
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - NEIMS
  - RDKit
  - gensim
  - hnswlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ei-mass-spectrum-library-matching

## Summary

Identify EI-MS compounds by embedding query spectra with a Word2vec spectral embedding model and performing HNSW approximate nearest-neighbor search against a million-scale in-silico library built from NEIMS-predicted spectra. This enables identification of molecules absent from experimental libraries such as NIST 2017.

## When to use

Use when measured GC-EI-MS spectra must be matched against a large in-silico spectral library, especially for compounds beyond NIST 2017, and exact nearest-neighbor search over millions of spectra is too slow.

## When NOT to use

- Input spectra were acquired on ionization modes other than electron ionization (EI) GC-MS, since FastEI's library and embedding model are trained on EI spectra
- The compound spectrum is already present in the NIST 2017 experimental library and a simple spectrum matching search suffices — FastEI's advantage targets molecules beyond the 0.27 million-compound NIST 2017 library
- You need an exact brute-force nearest-neighbor result rather than HNSW approximate search (approximate search may occasionally misrank, as with compound 10 ranked outside the top 100)

## Inputs

- Measured EI-MS query spectra files placed in the FastEI 'spectra' directory
- IN_SILICO_LIBRARY.db downloaded from https://zenodo.org/record/7293922
- references_index.bin (precomputed HNSW index of library spectral embeddings)
- references_word2vec.model (Word2vec spectral embedding model trained on 2,123,695 predicted spectra)

## Outputs

- Ranked list of nearest-neighbor library compounds (compound identification ranks) per query spectrum
- Per-compound rank table (e.g., identification_ranks.csv) with recall@1, recall@10, and recall@20 values

## How to apply

Place the query spectra files (e.g., the 10 measured EI-MS spectra of synthetic compounds) into the 'spectra' directory of the FastEI working tree, and place the downloaded IN_SILICO_LIBRARY.db (from Zenodo record 7293922), references_index.bin, and references_word2vec.model into the 'data' directory so the expanded library (~2,146,690 in-silico molecules from f-NIST and f-ChEMBL) and its precomputed HNSW index are available. Run test.py, which converts each query spectrum into a Word2vec spectral embedding and performs HNSW approximate nearest-neighbor search using hnswlib with M = 64, efConstruction = 600, and ef = 300 against the million-scale library. Extract the rank of the true compound per query, optionally apply a 5 Da mass filter to improve accuracy, and compute recall@1, recall@10, and recall@20. Validate against the reported benchmarks (recall@1 = 50%, recall@10 = 70%, recall@20 = 90% on the 10-compound extra test set) and save the per-compound rank table.

## Related tools

- **FastEI** (Standalone pipeline that transforms query EI spectra into Word2vec embeddings and performs HNSW library matching against the million-scale in-silico library) — https://github.com/Qiong-Yang/FastEI
- **hnswlib** (Approximate nearest-neighbor search engine over library spectral embeddings (M = 64, efConstruction = 600, ef = 300)) — https://github.com/nmslib/hnswlib
- **gensim** (Python library used to train the Word2vec model that embeds spectra as low-dimensional vectors for matching)
- **NEIMS** (Pre-trained model used (without re-training) to predict the in-silico mass spectra that populate the million-scale library) — https://github.com/brain-research/deep-molecular-massspec
- **RDKit** (Used (v2022.03.3) to read SMILES strings of molecules and compute ECFPs fed into NEIMS for library spectrum prediction)

## Examples

```
git clone https://github.com/Qiong-Yang/FastEI.git && cd FastEI && conda env create -f FastEI.yml && conda activate FastEI && python test.py
```

## Evaluation signals

- Rank table contains exactly 10 rows (one per measured query spectrum) when run on the extra test set
- Recall@1 = 50% (5/10), recall@10 = 70% (7/10), and recall@20 = 90% (9/10) match the published benchmark on the extra test set
- Only compound 10 — the compound with a 12-carbon long-chain substituent — is ranked beyond 100; all other true compounds rank within the top 20
- The HNSW index loads successfully with the expected parameters (M = 64, efConstruction = 600, ef = 300) and the search returns hits from the ~2,146,690-molecule in-silico library
- Applying the 5 Da mass filter should not degrade accuracy on the extra test set and improves recall@10 to 88.3% on the NIST-derived test set

## Limitations

- Recall@1 for compounds beyond NIST 2017 is only 50%, so top-1 identifications require manual verification; large molecules such as the 12-carbon long-chain compound (compound 10) may rank beyond the top 100
- The 10-molecule extra test set is a very small sample, so performance estimates carry high variance
- The Word2vec hyperparameter table (Supplementary Table 1) and detailed extra-test-set measurement conditions (Supplementary Note 2) are not in the main text, limiting full reproducibility from the paper alone
- FastEI's packaged GUI install version only supports 64-bit Windows (tested on Windows 7 and 10); Linux users must use the development conda environment

## Evidence

- [abstract] When FastEI is applied to identify the molecules beyond NIST 2017 library, it achieves 50% recall@1 accuracy.: "When FastEI is applied to identify the molecules beyond NIST 2017 library, it achieves 50% recall@1 accuracy."
- [discussion] HNSW search parameters used for library matching: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [discussion] recall metrics on the 10-compound extra test set: "its recall@1 is 50%, recall@10 is 70%, and recall@20 is 90% by FastEI, respectively"
- [methods] Zenodo download location for the in-silico library database: "Please put [IN_SILICO_LIBRARY.db](https://zenodo.org/record/7293922/files/IN_SILICO_LIBRARY.db)"
- [readme] FastEI repository README describing the method: "FastEI is an ultra-fast and accurate spectrum matching method, proposed to improve accuracy by Word2vec-based spectrum embedding and boost the speed using hierarchical navigable small world graph"
