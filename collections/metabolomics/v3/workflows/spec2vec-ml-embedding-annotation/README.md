# spec2vec-ml-embedding-annotation-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — raw MS2 spectra -> filtered, normalized spectrum objects  →  `mass-spectrometry-peak-list-processing`, `spectral-data-cleaning-with-matchms`, `matchms-spectral-filtering`, `mass-spectra-peak-filtering--7b28be9064fa4584`
2. **document_conversion** — filtered spectra -> peak/neutral-loss "documents" for word-embedding training  →  `spectral-peak-to-word-tokenization`, `msms-spectrum-document-conversion`, `spectra-to-document-conversion`, `peak-intensity-weighting-of-spectral-tokens`
3. **embed** — peak/loss documents -> Spec2Vec Word2Vec embedding model + query-spectrum embedding vectors  →  `spectral-word-embedding-vector-aggregation`, `word2vec-spectrum-embedding-training`, `gensim-word2vec-model-inference`, `spec2vec-spectral-embedding-scoring`
4. **retrieve_annotate** — query embeddings -> nearest-neighbour library matches -> embedding-based annotation table  →  `spectral-library-matching--599e7745b946f0de`, `spectral-embedding-model-training`, `unsupervised-spectral-word2vec-embedding-training`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
