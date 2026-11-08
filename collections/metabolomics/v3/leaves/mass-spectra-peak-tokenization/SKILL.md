---
name: mass-spectra-peak-tokenization
description: Use when an in-silico EI-MS library of predicted spectra (e.g. NEIMS
  predictions over f-NIST and f-ChEMBL) must be transformed into fixed-length spectral
  embeddings to enable approximate nearest-neighbor matching with HNSW.
when_to_use_negative:
- Input is raw GC-MS instrument data (e.g. Shimadzu QP2010Ultra acquisitions) rather
  than preprocessed peak-list spectra — tokenization presumes peak tokens are already
  derivable
- The matching task needs exact nearest-neighbor search on the full library (e.g.
  faiss.IndexFlatL2 weighted cosine similarity baseline) rather than the FastEI approximate
  HNSW pipeline
- Spectra come from a non-EI ionization method or a library not built from the NEIMS
  predicted-spectra pipeline; the embedding was trained on in-silico EI-MS from f-NIST/f-ChEMBL
  only
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: gensim
  role: Python library used to train and serialize the CBOW Word2vec model on the
    spectral peak-token corpus
  repo: https://github.com/Qiong-Yang/FastEI
- name: FastEI
  role: Pipeline that loads the trained Word2vec model to transform spectra into embeddings
    and feeds them to the hnswlib HNSW index
  repo: https://github.com/Qiong-Yang/FastEI
- name: NEIMS
  role: Pre-trained model that generated the predicted mass spectra (from ECFPs) that
    form the Word2vec training corpus
  repo: https://github.com/brain-research/deep-molecular-massspec
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:3bf9d79f7abca6e65ec130a7729cdc128c3609144ac0f41fefa547f881a6deab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - NEIMS
  - gensim
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# mass-spectra-peak-tokenization

## Summary

Reconstructs the FastEI Word2vec spectral embedding step from Yang et al. 2023 (Nature Communications), in which each predicted EI-MS spectrum is formatted as a 'sentence' of peak tokens and a gensim CBOW model (dim 500, window 1000, negative 5, lr 0.025 with linear decay, 60 epochs) is trained on 2,123,695 predicted spectra so spectra can be converted into dense embeddings for library matching.

## When to use

Use when an in-silico EI-MS library of predicted spectra (e.g. NEIMS predictions over f-NIST and f-ChEMBL) must be transformed into fixed-length spectral embeddings to enable approximate nearest-neighbor matching with HNSW.

## When NOT to use

- Input is raw GC-MS instrument data (e.g. Shimadzu QP2010Ultra acquisitions) rather than preprocessed peak-list spectra — tokenization presumes peak tokens are already derivable
- The matching task needs exact nearest-neighbor search on the full library (e.g. faiss.IndexFlatL2 weighted cosine similarity baseline) rather than the FastEI approximate HNSW pipeline
- Spectra come from a non-EI ionization method or a library not built from the NEIMS predicted-spectra pipeline; the embedding was trained on in-silico EI-MS from f-NIST/f-ChEMBL only

## Inputs

- Predicted EI mass spectra corpus: 2,123,695 spectra from f-NIST training/validation/test spectra and f-ChEMBL (in-silico library built from NEIMS predictions)
- Word2vec hyperparameters (Supplementary Table 1): CBOW, window 1000, dim 500, negative 5, lr 0.025, linear decay 0.00025, 60 epochs
- Python environment with gensim (e.g. FastEI conda env from FastEI.yml)

## Outputs

- Trained gensim Word2vec model serialized to disk (e.g. references_word2vec.model in the FastEI data directory) with 500-dimensional peak-token vectors
- Spectral embeddings obtained by summing peak embeddings, suitable for building the HNSW index

## How to apply

Load the predicted-spectra corpus (2,123,695 spectra from f-NIST and f-ChEMBL of the 2,146,690-molecule in-silico library, obtained by feeding ECFPs of the molecules into the NEIMS model) and format each spectrum as a 'sentence' of peak tokens for gensim Word2vec. Train gensim's Word2vec with the paper's hyperparameters from Supplementary Table 1: CBOW architecture, window size 1000, embedding dimension 500, negative sampling 5, initial learning rate 0.025 with linear decay 0.00025, for 60 epochs over the full spectral corpus. The rationale is that Word2vec embeddings capture peak co-occurrence structure, which the paper reports improves identification accuracy (80.4% recall@10; 88.3% with a 5 Da mass filter) compared to direct spectrum matching. Serialize the trained model (e.g. as references_word2vec.model in the data directory) so it can later transform both library and query spectra into low-dimensional embeddings by summing peak embeddings. Validate that the serialized model loads as a gensim Word2vec object whose vectors are 500-dimensional and whose configuration matches the Supplementary Table 1 hyperparameters.

## Related tools

- **gensim** (Python library used to train and serialize the CBOW Word2vec model on the spectral peak-token corpus) — https://github.com/Qiong-Yang/FastEI
- **FastEI** (Pipeline that loads the trained Word2vec model to transform spectra into embeddings and feeds them to the hnswlib HNSW index) — https://github.com/Qiong-Yang/FastEI
- **NEIMS** (Pre-trained model that generated the predicted mass spectra (from ECFPs) that form the Word2vec training corpus) — https://github.com/brain-research/deep-molecular-massspec

## Examples

```
conda activate FastEI; python -c "from gensim.models import Word2vec; m = Word2Vec(sentences=peak_token_sentences, sg=0, window=1000, vector_size=500, negative=5, alpha=0.025, min_alpha=0.00025, epochs=60); m.save('references_word2vec.model')"
```

## Evaluation signals

- references_word2vec.model exists in the data directory and loads as a valid gensim Word2vec model
- Model vectors are 500-dimensional
- Model configuration matches Supplementary Table 1: CBOW, window 1000, dim 500, negative 5, lr 0.025, decay 0.00025, 60 epochs
- Training corpus size matches the paper's 2,123,695 predicted spectra
- Downstream FastEI matching reaches the reported accuracy range (80.4% recall@10, or 88.3% with the 5 Da mass filter) on the test set

## Limitations

- The full hyperparameter listing (Supplementary Table 1) is referenced but not included in the main text; lr/decay values come from the reconstruction card
- Embedding quality depends on the in-silico library: it was trained on predicted spectra of f-NIST and f-ChEMBL molecules only, filtered by mass < 1000 Da, 11 common elements (H, C, O, N, P, S, Cl, F, Br, I, Si), non-ionic compounds, LogP in [-12, 24], and deduplication
- The pretrained NEIMS weights are downloaded and used without re-training, and their version/build date is not stated
- Performance on molecules beyond the NIST 2017 library was evaluated on a very small extra test set (10 synthetic compounds)

## Evidence

- [discussion] The Word2vec model was trained on the training set of 2,123,695 predicted spectra.: "It consists of 2,123,695 predicted spectra"
- [discussion] Spectra are transformed into low-dimensional embeddings by summing peak embeddings for HNSW indexing.: "all the predicted spectra were transformed into low-dimensional embeddings by summing the peak embedding to build the HNSW index for library matching"
- [discussion] gensim was the library used to train the Word2vec model.: "the Word2vec model was trained using the Word2vec training set based on gensim39, a Python library"
- [abstract] FastEI improves accuracy via Word2vec spectral embedding and speed via HNSW.: "substantially improve accuracy using Word2vec spectral embedding and boost the speed using the hierarchical navigable small-world graph (HNSW)"
- [readme] FastEI depends on gensim installable from conda-forge in the FastEI environment.: "conda install -c conda-forge gensim"
