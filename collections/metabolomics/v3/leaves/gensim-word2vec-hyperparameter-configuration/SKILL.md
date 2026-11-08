---
name: gensim-word2vec-hyperparameter-configuration
description: 'Use when reconstructing the Word2vec spectral embedding of FastEI: you
  have a corpus of predicted EI mass spectra formatted as peak-token sentences and
  need a 500-dim embedding model matching Supplementary Table 1.'
when_to_use_negative:
- You only need to query the pre-built FastEI library — the installation package already
  ships the Word2vec model and HNSW index, so no retraining is required
- Your goal is predicting spectra from molecular structures (ECFP inputs) — that is
  the NEIMS model, not Word2vec spectral embedding
- You need exact nearest-neighbor similarity over raw spectra (e.g., weighted cosine
  similarity via faiss.IndexFlatL2) rather than learned embedding-based approximate
  matching
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: gensim
  role: Python library used to train the Word2vec spectral embedding model over the
    peak-token sentences
  repo: https://github.com/Qiong-Yang/FastEI
- name: FastEI
  role: Consumes the serialized references_word2vec.model to transform spectra into
    embeddings; its matched results were used to optimize the Word2vec hyperparameters
  repo: https://github.com/Qiong-Yang/FastEI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39279-7
    title: Ultra-fast and accurate electron ionization mass spectrum matching for
      compound identification with million-scale in-silico library
  source_content_hash: sha256:8b35841036e5d5bc04caeb25b7adfb449d838b16625e59d9617261b8d14e505b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Qiong-Yang/FastEI
  tools:
  - FastEI
  - gensim
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# gensim-word2vec-hyperparameter-configuration

## Summary

Reconstructs the FastEI spectral embedding model by training a gensim Word2vec model (CBOW) on the paper's spectral 'sentences' of peak tokens with the published hyperparameters: window 1000, dimension 500, negative sampling 5, lr 0.025 with linear decay 0.00025, 60 epochs.

## When to use

Use when reconstructing the Word2vec spectral embedding of FastEI: you have a corpus of predicted EI mass spectra formatted as peak-token sentences and need a 500-dim embedding model matching Supplementary Table 1.

## When NOT to use

- You only need to query the pre-built FastEI library — the installation package already ships the Word2vec model and HNSW index, so no retraining is required
- Your goal is predicting spectra from molecular structures (ECFP inputs) — that is the NEIMS model, not Word2vec spectral embedding
- You need exact nearest-neighbor similarity over raw spectra (e.g., weighted cosine similarity via faiss.IndexFlatL2) rather than learned embedding-based approximate matching

## Inputs

- Corpus of 2,123,695 predicted EI mass spectra (f-NIST training/validation/test spectra plus f-ChEMBL from the in-silico library), each formatted as a 'sentence' of peak tokens for gensim
- Word2vec hyperparameters from Supplementary Table 1: CBOW, window 1000, dim 500, negative 5, lr 0.025, decay 0.00025, 60 epochs

## Outputs

- Trained gensim Word2vec model serialized as references_word2vec.model (500-dimensional peak/spectral vectors)
- Validation report confirming model dimensionality and hyperparameter configuration match Supplementary Table 1

## How to apply

Format each of the 2,123,695 predicted spectra (from f-NIST and f-ChEMBL of the in-silico library) as a 'sentence' of peak tokens for gensim Word2vec. Configure gensim's Word2vec with the CBOW architecture (sg=0), window size 1000, embedding dimension 500, negative sampling 5, initial learning rate 0.025 with linear decay 0.00025, and train for 60 full epochs over the spectral corpus. The paper states hyperparameters were optimized on a validation set according to FastEI's matched results, so if re-tuning, rank candidates by FastEI recall rather than intrinsic Word2vec metrics. Serialize the trained model to disk as references_word2vec.model in the data directory so FastEI can load it to transform spectra into embeddings (by summing peak embeddings) for HNSW indexing. Validate that the saved model loads as a gensim Word2vec model with 500-dimensional vectors whose configuration matches Supplementary Table 1.

## Related tools

- **gensim** (Python library used to train the Word2vec spectral embedding model over the peak-token sentences) — https://github.com/Qiong-Yang/FastEI
- **FastEI** (Consumes the serialized references_word2vec.model to transform spectra into embeddings; its matched results were used to optimize the Word2vec hyperparameters) — https://github.com/Qiong-Yang/FastEI

## Examples

```
from gensim.models import Word2Vec; model = Word2Vec(spectra_sentences, sg=0, window=1000, vector_size=500, negative=5, alpha=0.025, min_alpha=0.00025, epochs=60); model.save('references_word2vec.model')
```

## Evaluation signals

- references_word2vec.model exists in the data directory and loads as a valid gensim Word2vec model
- All word vectors in the loaded model have exactly 500 dimensions
- Model hyperparameters match Supplementary Table 1: CBOW, window 1000, dim 500, negative 5, lr 0.025, decay 0.00025, 60 epochs
- Training iterated over all 2,123,695 predicted spectra for the full 60 epochs
- Downstream FastEI spectral matching reproduces the reported accuracy behavior (80.4% recall@10, 88.3% with 5 Da mass filter) on the test set

## Limitations

- The full hyperparameter table (Supplementary Table 1) is referenced but not included in the article text, so exact values beyond those stated rely on the SI
- Hyperparameter optimization was driven by FastEI matched results on a validation set, not intrinsic Word2vec quality metrics
- Performance claims on the 10-molecule extra test set (recall@1 50%) rest on a very small sample
- FastEI's packaged GUI is Windows 64-bit only (tested on Windows 7/10); the conda/gensim route is the development path on other systems

## Evidence

- [discussion] the Word2vec model was trained using the Word2vec training set based on gensim39, a Python library: "the Word2vec model was trained using the Word2vec training set based on gensim39, a Python library"
- [discussion] It consists of 2,123,695 predicted spectra: "It consists of 2,123,695 predicted spectra"
- [discussion] It was used to optimize the hyperparameters of the Word2vec model according to its matched results of FastEI: "It was used to optimize the hyperparameters of the Word2vec model according to its matched results of FastEI"
- [discussion] All the hyperparameters of Word2vec are listed in Supplementary Table 1: "All the hyperparameters of Word2vec are listed in Supplementary Table 1"
- [readme] conda install -c conda-forge gensim: "conda install -c conda-forge gensim"
