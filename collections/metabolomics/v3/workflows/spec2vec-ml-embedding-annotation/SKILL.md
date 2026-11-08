---
name: spec2vec-ml-embedding-annotation-workflow
description: 'Use when you want to annotate untargeted MS2 spectra with a machine-learned
  spectral embedding instead of raw cosine scoring — convert spectra into peak/neutral-loss
  "documents", train or load a Spec2Vec (Word2Vec-style) embedding model on a reference
  corpus, embed query spectra into that learned vector space, and retrieve/annotate
  nearest library neighbours by embedding similarity, as a deep-learning-adjacent
  second pass beyond modified-cosine matching.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - mass-spectrometry-peak-list-processing
  - spectral-data-cleaning-with-matchms
  - matchms-spectral-filtering
  - mass-spectra-peak-filtering--7b28be9064fa4584
  - spectral-peak-to-word-tokenization
  - msms-spectrum-document-conversion
  - spectra-to-document-conversion
  - peak-intensity-weighting-of-spectral-tokens
  - spectral-word-embedding-vector-aggregation
  - word2vec-spectrum-embedding-training
  - gensim-word2vec-model-inference
  - spec2vec-spectral-embedding-scoring
  - spectral-library-matching--599e7745b946f0de
  - spectral-embedding-model-training
  - unsupervised-spectral-word2vec-embedding-training
  member_tools:
  - matchms
  - pymzml
  - pyteomics
  - spectrum_utils
  - spec2vec
  - gensim
  - MS2DECIDE
  - gensim (Word2Vec)
  - spec2vec_gnps_data_analysis
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Spec2Vec / ML Spectral-Embedding Annotation (peak-loss documents -> Word2Vec embedding -> nearest-neighbour ID)

## Summary

MS2 in, an embedding-based annotation table out: peak/loss document conversion, Spec2Vec (Word2Vec) model training or loading, query-spectrum embedding, and nearest-neighbour library retrieval and annotation.


## When to use

Use when you want to annotate untargeted MS2 spectra with a machine-learned spectral embedding instead of raw cosine scoring — convert spectra into peak/neutral-loss "documents", train or load a Spec2Vec (Word2Vec-style) embedding model on a reference corpus, embed query spectra into that learned vector space, and retrieve/annotate nearest library neighbours by embedding similarity, as a deep-learning-adjacent second pass beyond modified-cosine matching.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** raw MS2 spectra -> filtered, normalized spectrum objects

**EDAM operation:** http://edamontology.org/operation_3630

**Inputs:** mzML · **Outputs:** mgf/sirius

**Candidate leaf skills:** `mass-spectrometry-peak-list-processing` (primary), `spectral-data-cleaning-with-matchms`, `matchms-spectral-filtering`, `mass-spectra-peak-filtering--7b28be9064fa4584`

**Tools (primary):** matchms, pymzml, pyteomics, spectrum_utils

**Other candidate tools:** MS2Query, R, Spectra, SpectriPy, reticulate, MS2DeepScore

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-023-37446-4, 10.1186/s13321-021-00558-4, 10.21105/joss.02411, 10.21105/joss.08070

### Stage 2 — document_conversion

**Goal:** filtered spectra -> peak/neutral-loss "documents" for word-embedding training

**EDAM operation:** http://edamontology.org/operation_3357

**Inputs:** mgf/sirius · **Outputs:** json/spectral-documents

**Candidate leaf skills:** `spectral-peak-to-word-tokenization` (primary), `msms-spectrum-document-conversion`, `spectra-to-document-conversion`, `peak-intensity-weighting-of-spectral-tokens`

**Tools (primary):** matchms, spec2vec

**Other candidate tools:** Word2Vec (gensim), gensim

**Grounding:** 1 KB(s); DOIs: 10.1371/journal.pcbi.1008724

### Stage 3 — embed

**Goal:** peak/loss documents -> Spec2Vec Word2Vec embedding model + query-spectrum embedding vectors

**EDAM operation:** http://edamontology.org/operation_3800

**Inputs:** json/spectral-documents · **Outputs:** json/spectrum-embeddings

**Candidate leaf skills:** `spectral-word-embedding-vector-aggregation` (primary), `word2vec-spectrum-embedding-training`, `gensim-word2vec-model-inference`, `spec2vec-spectral-embedding-scoring`

**Tools (primary):** gensim, matchms, spec2vec

**Other candidate tools:** Word2Vec, spec2vec_gnps_data_analysis, Word2Vec (gensim)

**Grounding:** 1 KB(s); DOIs: 10.1371/journal.pcbi.1008724

### Stage 4 — retrieve_annotate

**Goal:** query embeddings -> nearest-neighbour library matches -> embedding-based annotation table

**EDAM operation:** http://edamontology.org/operation_3767

**Inputs:** json/spectrum-embeddings · **Outputs:** tsv

**Candidate leaf skills:** `spectral-library-matching--599e7745b946f0de` (primary), `spectral-embedding-model-training`, `unsupervised-spectral-word2vec-embedding-training`

**Tools (primary):** MS2DECIDE, Spec2Vec, gensim (Word2Vec), matchms, spec2vec_gnps_data_analysis

**Other candidate tools:** gensim, GNPS, MS2Query, scikit-learn

**Grounding:** 2 KB(s); DOIs: 10.1038/s41467-023-37446-4, 10.1371/journal.pcbi.1008724

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
