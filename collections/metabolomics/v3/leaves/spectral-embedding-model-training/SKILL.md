---
name: spectral-embedding-model-training
description: 'Use when you have a large curated MS/MS library (e.g. GNPS-derived AllPositive:
  95,320 positive-mode spectra, 77,092 InChIKey-annotated) and need a learned spectral
  similarity model for library matching or molecular networking beyond plain cosine
  scoring.'
when_to_use_negative:
- When no suitable training data exist and a training-free score is needed — Spec2Vec,
  unlike cosine scores, requires training data to learn fragment peak relationships
- When the query data contain many fragment/loss features absent from the training
  set (missing-word issue), e.g. models trained on smaller datasets or with more m/z
  decimals included in words
- For GC-MS data — performance was only demonstrated on LC-MS data and GC-MS utility
  was not assessed
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3520
tools:
- name: spec2vec
  role: Python package implementing the Word2Vec-inspired spectral embedding model,
    spectrum-vector computation and Spec2Vec similarity score
  repo: https://github.com/iomega/spec2vec
- name: gensim
  role: Trains the Word2Vec model on documents of peak and loss words
- name: matchms
  role: Imports, cleans and corrects spectra metadata and peaks before document conversion
    and model training
  repo: https://github.com/matchms/matchms
- name: spec2vec_gnps_data_analysis
  role: Notebook repository with the analysis and benchmarking pipeline that trained
    models on GNPS data
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:0c6e2800ef1e722f353cc377e207f69fe9c66ab7df73ac57e1a19ca98c33eab2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - gensim
  - matchms
  - spec2vec
  - spec2vec_gnps_data_analysis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# spectral-embedding-model-training

## Summary

Trains a Word2Vec (gensim) model on mass spectra converted to documents of peak and neutral-loss words, producing learned spectral embeddings whose cosine similarity serves as the Spec2Vec spectral similarity score. It exists because learnt embeddings correlate better with structural (Tanimoto) similarity than cosine-based scores and scale to large library searches.

## When to use

Use when you have a large curated MS/MS library (e.g. GNPS-derived AllPositive: 95,320 positive-mode spectra, 77,092 InChIKey-annotated) and need a learned spectral similarity model for library matching or molecular networking beyond plain cosine scoring.

## When NOT to use

- When no suitable training data exist and a training-free score is needed — Spec2Vec, unlike cosine scores, requires training data to learn fragment peak relationships
- When the query data contain many fragment/loss features absent from the training set (missing-word issue), e.g. models trained on smaller datasets or with more m/z decimals included in words
- For GC-MS data — performance was only demonstrated on LC-MS data and GC-MS utility was not assessed

## Inputs

- Processed positive-ionization MS/MS spectra (e.g. from GNPS ALL_GNPS.json, cleaned with matchms: peaks restricted to m/z 0–1000, ≥10 peaks per spectrum)
- Spectra converted to documents of peak words and loss words ('loss@xxx.xx', neutral losses 5.0–200.0 Da)
- Optionally InChIKey-annotated spectra for the UniqueInchiKeys (12,797 spectra) or AllPositive (95,320 spectra) dataset variants

## Outputs

- Trained gensim Word2Vec model of learned fragment/loss word embeddings (e.g. Zenodo records 3979074 / 4173596)
- Intensity-weighted spectrum vectors vS = Σ wi·vi for each spectrum
- Spec2Vec cosine similarity scores between spectra, usable for library matching and molecular networking

## How to apply

First clean spectra with matchms (remove peaks with m/z outside [0, 1000], discard spectra with fewer than 10 peaks), then convert every spectrum into a document where each peak becomes a word encoding its m/z position at a fixed decimal precision and neutral losses between 5.0 and 200.0 Da are added as 'loss@xxx.xx' words. Train a Word2Vec model on all documents using gensim (pre-trained models for the AllPositive and UniqueInchiKeys datasets are available on Zenodo records 4173596 and 3979074). Compute each spectrum vector vS as the intensity-weighted sum of its word vectors; the Spec2Vec score is the cosine between two spectrum vectors. When scoring spectra far from the training distribution, enforce a missing-fraction threshold (e.g. <0.05) so that query spectra dominated by words unseen during training do not receive misleading scores. Note the training set must contain a large fraction of the fragment/loss features expected in future query data, so retrain (or add new spectra) when coverage is low.

## Related tools

- **spec2vec** (Python package implementing the Word2Vec-inspired spectral embedding model, spectrum-vector computation and Spec2Vec similarity score) — https://github.com/iomega/spec2vec
- **gensim** (Trains the Word2Vec model on documents of peak and loss words)
- **matchms** (Imports, cleans and corrects spectra metadata and peaks before document conversion and model training) — https://github.com/matchms/matchms
- **spec2vec_gnps_data_analysis** (Notebook repository with the analysis and benchmarking pipeline that trained models on GNPS data) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Examples

```
conda install --channel nlesc --channel bioconda --channel conda-forge spec2vec
```

## Evaluation signals

- A trained model's quality can be checked by the unknown-compound benchmark: for ~60% of 1030 held-out queries the top-10 matches should contain a molecule with Tanimoto structural similarity > 0.6 (random-draw p = 0.0103)
- Per-query runtime for structural-analogue search against 76,062 library spectra should stay near ~0.14 s (~140 s total)
- Missing-fraction check: spectra whose words are largely outside the model vocabulary must be flagged/excluded by the <0.05 missing-fraction threshold rather than scored
- Pre-trained models must match the dataset variant (AllPositive vs UniqueInchiKeys) — verify against Zenodo records 4173596 (AllPositive) and 3979074 / doi:10.5281/zenodo.3978054 (UniqueInchikey)
- Sanity check that learned word embeddings make related fragments/losses point in similar vector directions (the Word2Vec analogy to word relationships)

## Limitations

- Spec2Vec needs training data to learn fragment peak relationships, unlike cosine scores
- The training set must include a large fraction of the fragment ions and losses seen in query data; missing words become relevant with smaller training sets or more m/z decimals per word
- Network analysis or other applications may require a massive training set or retraining including new experimental spectra
- Performance demonstrated only on LC-MS data; GC-MS utility not assessed
- Some remaining peaks/losses not covered by the model may not be chemically meaningful (e.g. many low-Dalton values)

## Evidence

- [discussion] A Word2Vec [22] model is trained on all documents of a chosen dataset using gensim [37].: "A Word2Vec [22] model is trained on all documents of a chosen dataset using gensim [37]."
- [discussion] In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as "loss@xxx.xx".: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [discussion] By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided.: "By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided."
- [readme] Spec2vec is a novel spectral similarity score inspired by a natural language processing algorithm -- Word2Vec: "**Spec2vec** is a novel spectral similarity score inspired by a natural language processing
algorithm -- Word2Vec."
- [discussion] spec2vec needs training data to learn the fragment peak relationships: "one limitation of Spec2Vec as compared to cosine scores is that it needs training data to learn the fragment peak relationships"
