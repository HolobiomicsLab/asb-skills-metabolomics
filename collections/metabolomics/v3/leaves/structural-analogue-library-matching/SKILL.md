---
name: structural-analogue-library-matching
description: Use when unknown query spectra must be matched to a large spectral library
  (e.g. GNPS ALL_GNPS.json) to detect highly related molecules via Spec2Vec similarity
  scores.
when_to_use_negative:
- The goal is exact library matching of already-known compounds rather than detecting
  structural analogues among unknowns
- The input data is GC-MS spectra — performance was only demonstrated on LC-MS data
- 'No adequate training data exists: Spec2Vec needs training data to learn fragment
  peak relationships, unlike cosine scores'
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Spec2Vec
  role: Computes spectral embeddings and cosine similarity scores for analogue matching
  repo: https://github.com/iomega/spec2vec
- name: Word2Vec (gensim)
  role: Trains the underlying embedding model on documents of peak and loss words
- name: matchms
  role: Cleans, corrects metadata, and processes spectra before matching
  repo: https://github.com/matchms/matchms
- name: spec2vec_gnps_data_analysis
  role: Reference notebooks and benchmarking scripts reproducing this workflow
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:d44b3fa46efc6e294e472fd0b331bf758da8971033a6da0271e9977ab5b706fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - Spec2Vec
  - Word2Vec (gensim)
  - matchms
  - spec2vec_gnps_data_analysis
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# structural-analogue-library-matching

## Summary

Matches unknown query MS/MS spectra against a large library using Spec2Vec embeddings to retrieve top-10 structural analogues, verified by Tanimoto similarity against held-out InChIKeys.

## When to use

Use when unknown query spectra must be matched to a large spectral library (e.g. GNPS ALL_GNPS.json) to detect highly related molecules via Spec2Vec similarity scores.

## When NOT to use

- The goal is exact library matching of already-known compounds rather than detecting structural analogues among unknowns
- The input data is GC-MS spectra — performance was only demonstrated on LC-MS data
- No adequate training data exists: Spec2Vec needs training data to learn fragment peak relationships, unlike cosine scores

## Inputs

- Processed AllPositive dataset of positive-ionization MS/MS spectra (95,320 spectra from GNPS ALL_GNPS.json, 77,092 with InChIKey annotations)
- Held-out query set of spectra from 200 randomly chosen InChIKeys (1030 spectra)
- Trained Word2Vec/Spec2Vec model (e.g. zenodo.org/record/4173596, trained on AllPositive dataset)

## Outputs

- Top-10 candidate library matches per query spectrum with Spec2Vec cosine similarity scores
- Tanimoto structural similarity of each suggested match to the query molecule
- Per-query runtimes (reported ~0.14 s per query, ~140 s total)

## How to apply

Split the processed AllPositive dataset (95,320 spectra, 77,092 with InChIKey annotations, derived from GNPS ALL_GNPS.json) by randomly selecting 200 InChIKeys (1030 spectra) as a held-out query set, and remove all spectra without InChIKey annotation so training (76,062 spectra) and query data share no molecules. Convert training spectra to documents of peak and loss words (neutral losses between 5.0 and 200.0 Da) and train a Word2Vec model with gensim. Pre-select candidate matches by precursor m/z with 1 ppm tolerance, then compute Spec2Vec cosine similarities between query vectors (intensity-weighted sums of word vectors) and candidate library vectors, retrieving the top-10 matches per query with their Tanimoto structural similarities. Pre-selection by precursor m/z before scoring is essential for scaling to large databases.

## Related tools

- **Spec2Vec** (Computes spectral embeddings and cosine similarity scores for analogue matching) — https://github.com/iomega/spec2vec
- **Word2Vec (gensim)** (Trains the underlying embedding model on documents of peak and loss words)
- **matchms** (Cleans, corrects metadata, and processes spectra before matching) — https://github.com/matchms/matchms
- **spec2vec_gnps_data_analysis** (Reference notebooks and benchmarking scripts reproducing this workflow) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Examples

```
conda install --channel nlesc --channel bioconda --channel conda-forge spec2vec; jupyter notebook  # run spec2vec_gnps_data_analysis notebooks
```

## Evaluation signals

- For ~60% of the 1030 query spectra, the top-10 suggested matches contain a molecule with Tanimoto structural similarity > 0.6
- This hit rate must far exceed the random-draw probability of p = 0.0103
- Average per-query runtime is ~0.14 s (~140 s total for all 1030 queries)
- Training and query sets share no molecule overlap: all spectra without InChIKey annotations are removed from the training set

## Limitations

- Spec2Vec requires training data, unlike cosine scores; the training set must include a large fraction of the fragment/loss features present in the query data
- Missing words become relevant when models are trained on smaller datasets or more m/z decimals are included; a missing-fraction threshold (e.g. <0.05) can avoid scoring spectra far outside the learned peaks
- Performance was only demonstrated on LC-MS data; GC-MS utility not assessed
- Other applications such as network analysis may require a massive training set or retraining including new experimental spectra

## Evidence

- [intro] To test whether Spec2Vec similarity scores can be used to help detect highly related molecules: "To test whether Spec2Vec similarity scores can be used to help detect highly related molecules, we moved all spectra belonging to 200 randomly chosen InChIKey (1030 spectra in total) from the"
- [intro] pre-selected based on precursor-m/z matches: "potentially matching spectra were pre-selected based on precursor-m/z matches (tolerance = 1ppm) before the highest scoring candidate above a similarity threshold was chosen."
- [intro] removed all spectra without InChIKey annotation: "To make sure that there is no overlap regarding molecules between training and query data, we also removed all spectra without InChIKey annotation."
- [discussion] neutral losses between 5.0 and 200.0 Da were added: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] A Word2Vec model is trained on all documents using gensim: "A Word2Vec [22] model is trained on all documents of a chosen dataset using gensim [37]."
- [abstract] Spec2Vec is computationally more scalable: "Spec2Vec is computationally more scalable allowing structural analogue searches in large databases within seconds."
- [readme] matchms and spec2vec packages: "matchms (https://github.com/matchms/matchms) and spec2vec (https://github.com/iomega/spec2vec)"
