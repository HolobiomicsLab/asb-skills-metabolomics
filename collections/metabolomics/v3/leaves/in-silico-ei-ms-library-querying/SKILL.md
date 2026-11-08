---
name: in-silico-ei-ms-library-querying
description: Use when you need to attribute FastEI's recall and throughput gains to
  Word2vec spectral embedding versus HNSW indexing, by benchmarking the EC and BH
  hybrid variants against the full FastEI pipeline and the WCS baseline.
when_to_use_negative:
- You only need end-to-end compound identification results and are not attributing
  performance to specific FastEI components — use the full FastEI pipeline instead
  of the EC/BH hybrids.
- Your query spectra are not EI mass spectra comparable to the NIST 2017 test set
  (e.g. MS/MS or LC-MS data) — the HNSW index and Word2vec model are trained on EI
  predicted spectra.
- You need exact nearest-neighbor ground truth on the million-scale library without
  approximation — HNSW (BH configuration) is an approximate nearest-neighbor method.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: FastEI
  role: Spectrum matching method whose Word2vec embedding and HNSW components are
    being ablated; provides the test harness (test.py) for recall and timing metrics
  repo: https://github.com/Qiong-Yang/FastEI
- name: gensim
  role: Python library in which the Word2vec model was trained; used to transform
    test spectra into low-dimensional embeddings for the EC configuration
- name: faiss.IndexFlatL2
  role: Exhaustive L2 index used in the EC configuration to compute cosine similarity
    between normalized spectral embeddings
- name: hnswlib
  role: Provides the prebuilt HNSW approximate nearest-neighbor index (M = 64, efConstruction
    = 600, ef = 300) queried in the BH configuration
  repo: https://github.com/nmslib/hnswlib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:d706095d0e7e35c0936872e5ba76537e610c8004c654adcf9d1b867d0c104553
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - faiss.IndexFlatL2
  - gensim
  - hnswlib
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# in-silico-ei-ms-library-querying

## Summary

Ablation protocol that decomposes FastEI's contribution by testing two hybrid configurations on measured NIST 2017 EI-MS test spectra: EC (Word2vec embeddings + cosine similarity via faiss.IndexFlatL2, replacing WCS binning) and BH (weighted spectral binning + HNSW, replacing cosine similarity). It isolates which component drives identification accuracy versus matching speed.

## When to use

Use when you need to attribute FastEI's recall and throughput gains to Word2vec spectral embedding versus HNSW indexing, by benchmarking the EC and BH hybrid variants against the full FastEI pipeline and the WCS baseline.

## When NOT to use

- You only need end-to-end compound identification results and are not attributing performance to specific FastEI components — use the full FastEI pipeline instead of the EC/BH hybrids.
- Your query spectra are not EI mass spectra comparable to the NIST 2017 test set (e.g. MS/MS or LC-MS data) — the HNSW index and Word2vec model are trained on EI predicted spectra.
- You need exact nearest-neighbor ground truth on the million-scale library without approximation — HNSW (BH configuration) is an approximate nearest-neighbor method.

## Inputs

- IN_SILICO_LIBRARY.db (million-scale in-silico EI-MS library, Zenodo record 7293922)
- references_word2vec.model (trained gensim Word2vec spectral embedding model)
- references_index.bin (prebuilt hnswlib HNSW index, M = 64, efConstruction = 600, ef = 300)
- Test-set spectra (over ten thousand measured NIST 2017 spectra)

## Outputs

- hybrid_matching_results.csv with per-configuration recall@1, recall@10, recall@20, and per-query matching time
- Reproduction of the Fig. 2d ablation comparison (EC, BH vs FastEI and WCS)

## How to apply

Download IN_SILICO_LIBRARY.db, references_word2vec.model, and references_index.bin from Zenodo record 7293922 and place them in the data directory. Run the EC configuration by transforming over ten thousand measured NIST 2017 test spectra with the trained gensim Word2vec model and computing cosine similarity between normalized embeddings via faiss.IndexFlatL2 (Euclidean distance between normalized vectors). Run the BH configuration by applying weighted spectral binning to query spectra and querying the prebuilt hnswlib HNSW index built with M = 64, efConstruction = 600, ef = 300. Compute recall@1, recall@10, recall@20, and per-query matching time with the FastEI test harness (test.py), then compare each hybrid against the full FastEI pipeline and the WCS baseline. The expected division of labor: EC matches FastEI-level accuracy but only at WCS-like speed, while BH improves speed partially at unchanged WCS-level accuracy; FastEI combining both is best on both axes.

## Related tools

- **FastEI** (Spectrum matching method whose Word2vec embedding and HNSW components are being ablated; provides the test harness (test.py) for recall and timing metrics) — https://github.com/Qiong-Yang/FastEI
- **gensim** (Python library in which the Word2vec model was trained; used to transform test spectra into low-dimensional embeddings for the EC configuration)
- **faiss.IndexFlatL2** (Exhaustive L2 index used in the EC configuration to compute cosine similarity between normalized spectral embeddings)
- **hnswlib** (Provides the prebuilt HNSW approximate nearest-neighbor index (M = 64, efConstruction = 600, ef = 300) queried in the BH configuration) — https://github.com/nmslib/hnswlib

## Examples

```
git clone https://github.com/Qiong-Yang/FastEI.git; conda env create -f FastEI.yml; conda activate FastEI; python test.py  # EC: faiss.IndexFlatL2 on Word2vec embeddings; BH: hnswlib index (M=64, ef=300)
```

## Evaluation signals

- EC recall@10 falls within the FastEI-reported range (~80.4%, or 88.3% with 5 Da mass filter) while its per-query speed remains at WCS-like levels
- BH accuracy matches the WCS baseline (unchanged) while showing a measurable per-query speed improvement over WCS
- Full FastEI outperforms both hybrids on both recall and spectra/second, reaching recall@1 of 36.7% at 238 spectra/second
- recall@1, recall@10, and recall@20 and per-query matching time are computed for all four configurations (FastEI, WCS, EC, BH) and written to hybrid_matching_results.csv
- Consistency with the Fig. 2d ablation pattern: embeddings drive accuracy, HNSW drives speed

## Limitations

- The BH configuration improves speed only partially relative to full FastEI because weighted spectral binning representations limit accuracy to WCS levels.
- The EC configuration is exact (IndexFlatL2) and therefore does not demonstrate the throughput gains that HNSW provides; speed conclusions require the BH arm.
- FastEI's standalone GUI version only supports Windows 64-bit (tested on Windows 7 and Windows 10); the development path requires a conda environment built from FastEI.yml.
- Recall figures depend on the specific test set (over ten thousand measured NIST 2017 spectra); performance on molecules beyond NIST 2017 is lower (50% recall@1).

## Evidence

- [abstract] FastEI improves accuracy using Word2vec spectral embedding and boosts speed using HNSW: "substantially improve accuracy using Word2vec spectral embedding and boost the speed using the hierarchical navigable small-world graph (HNSW)"
- [abstract] FastEI achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS): "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [discussion] Cosine similarity between normalized embeddings is computed via faiss.IndexFlatL2: "They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL241"
- [discussion] HNSW index built with hnswlib at specific hyperparameters: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [methods] Data files and FastEI environment provided in the repository: "Please put [IN_SILICO_LIBRARY.db](https://zenodo.org/record/7293922/files/IN_SILICO_LIBRARY.db)"
- [readme] Conda environment setup from FastEI.yml: "conda env create -f FastEI.yml 
      conda activate FastEI"
