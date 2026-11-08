---
name: mass-spectrum-matching-speed-profiling
description: Use when you need to determine which component of a spectrum-matching
  pipeline (embedding representation vs. approximate nearest-neighbor index) drives
  recall@k accuracy versus query throughput, as verifiable by testing hybrid configurations
  on measured spectra.
when_to_use_negative:
- You only need end-to-end FastEI identification accuracy on new compounds beyond
  NIST 2017 (e.g. the 10-molecule extra test set) rather than component attribution
- Your library is small enough that exact nearest-neighbor search is affordable —
  the HNSW speed dimension of the ablation is then irrelevant
- Your spectra are not electron-ionization spectra compatible with the Word2vec spectral
  embedding trained on 2,123,695 predicted spectra
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FastEI
  role: Provides the test harness (test.py), the full pipeline to compare against,
    and the WCS baseline
  repo: https://github.com/Qiong-Yang/FastEI
- name: gensim
  role: Trains/loads the Word2vec model used to transform query spectra into low-dimensional
    embeddings for the EC configuration
- name: faiss.IndexFlatL2
  role: Computes Euclidean distance between normalized embeddings (cosine similarity)
    for the EC configuration
- name: hnswlib
  role: Serves the prebuilt HNSW approximate nearest-neighbor index queried by the
    BH configuration (M = 64, efConstruction = 600, ef = 300)
  repo: https://github.com/nmslib/hnswlib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:9bde474d720e5dcd0f949b1080045deb8ac3d58f9c2815b452cbc6a52d895bec
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mass-spectrum-matching-speed-profiling

## Summary

Ablation-profiling skill for EI mass spectrum matching: it runs the hybrid configurations EC (Word2vec embeddings + cosine similarity) and BH (weighted binning + HNSW) against the full FastEI pipeline and the WCS baseline to attribute FastEI's accuracy gains to embeddings and its speed gains to the HNSW graph index.

## When to use

Use when you need to determine which component of a spectrum-matching pipeline (embedding representation vs. approximate nearest-neighbor index) drives recall@k accuracy versus query throughput, as verifiable by testing hybrid configurations on measured spectra.

## When NOT to use

- You only need end-to-end FastEI identification accuracy on new compounds beyond NIST 2017 (e.g. the 10-molecule extra test set) rather than component attribution
- Your library is small enough that exact nearest-neighbor search is affordable — the HNSW speed dimension of the ablation is then irrelevant
- Your spectra are not electron-ionization spectra compatible with the Word2vec spectral embedding trained on 2,123,695 predicted spectra

## Inputs

- IN_SILICO_LIBRARY.db (million-scale in-silico EI-MS library, 2,146,690 molecules, from Zenodo record 7293922)
- references_word2vec.model (trained Word2vec spectral embedding model)
- references_index.bin (prebuilt hnswlib HNSW index)
- Test-set spectra: over ten thousand measured NIST 2017 spectra

## Outputs

- hybrid_matching_results.csv with per-configuration recall@1, recall@10, recall@20 and per-query matching time
- Component attribution: EC accuracy/speed profile vs. BH accuracy/speed profile vs. FastEI and WCS baselines (Fig. 2d analogue)

## How to apply

Download IN_SILICO_LIBRARY.db, references_word2vec.model, and references_index.bin from Zenodo record 7293922 into the data directory. Transform the test-set spectra (over ten thousand measured NIST 2017 spectra) with the trained Word2vec model and run the EC configuration — cosine similarity between normalized embeddings computed as Euclidean distance with faiss.IndexFlatL2. Then run the BH configuration — apply weighted spectral binning to queries and search the prebuilt HNSW index built with hnswlib (M = 64, efConstruction = 600, ef = 300). Compute recall@1, recall@10, recall@20 and per-query matching time for each configuration with the FastEI test harness, then compare against full FastEI (recall@1 36.7% at 238 spectra/second) and the WCS baseline. Accept the result when EC matches FastEI-level accuracy at WCS-like speed and BH retains WCS-level accuracy with a measurable speed improvement, consistent with Fig. 2d.

## Related tools

- **FastEI** (Provides the test harness (test.py), the full pipeline to compare against, and the WCS baseline) — https://github.com/Qiong-Yang/FastEI
- **gensim** (Trains/loads the Word2vec model used to transform query spectra into low-dimensional embeddings for the EC configuration)
- **faiss.IndexFlatL2** (Computes Euclidean distance between normalized embeddings (cosine similarity) for the EC configuration)
- **hnswlib** (Serves the prebuilt HNSW approximate nearest-neighbor index queried by the BH configuration (M = 64, efConstruction = 600, ef = 300)) — https://github.com/nmslib/hnswlib

## Examples

```
git clone https://github.com/Qiong-Yang/FastEI.git && conda env create -f FastEI.yml && conda activate FastEI && python test.py
```

## Evaluation signals

- EC recall@10 falls within the FastEI-reported range (~80.4%, or 88.3% with a 5 Da mass filter) while its throughput is only WCS-like
- BH accuracy matches the WCS level while showing a measurable per-query speed improvement over WCS
- Full FastEI remains best in both speed and accuracy, consistent with recall@1 of 36.7% at 238 spectra/second
- hybrid_matching_results.csv contains recall@1, recall@10, recall@20 and per-query matching time rows for every tested configuration (EC, BH, FastEI, WCS)
- Index parameters used for BH match hnswlib settings M = 64, efConstruction = 600, ef = 300

## Limitations

- FastEI's packaged software currently supports only Windows 64-bit (tested on Windows 7 and Windows 10), constraining reproducibility environments
- Performance claims on the extra test set rest on only 10 measured spectra; the component attribution here relies on the NIST 2017 test set
- The WCS baseline is referenced for comparison but not defined in the visible text, complicating exact reimplementation
- The HNSW query-time/accuracy trade-off depends on the ef parameter (set to 300), which must be re-applied manually after loading the index

## Evidence

- [abstract] FastEI achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS).: "It achieves 80.4% recall@10 accuracy (88.3% with 5 Da mass filter) with a speedup of two orders of magnitude compared with the weighted cosine similarity method (WCS)."
- [abstract] Word2vec spectral embedding and HNSW each contribute distinct improvements to FastEI: "substantially improve accuracy using Word2vec spectral embedding and boost the speed using the hierarchical navigable small-world graph (HNSW)"
- [discussion] faiss.IndexFlatL2 used for embedding similarity in EC configuration: "They can be calculated by the Euclidean distance between two normalized vectors using faiss.IndexFlatL241"
- [discussion] hnswlib parameters for the HNSW index used by BH configuration: "using hnswlib (https://github.com/nmslib/hnswlib) with M = 64, efConstruction = 600, and ef = 300"
- [readme] FastEI improves accuracy via spectrum embedding and speed via HNSW, per README: "FastEI is an ultra-fast and accurate spectrum matching method, proposed to improve accuracy by Word2vec-based spectrum embedding and boost the speed using hierarchical navigable small world graph"
