---
name: word2vec-spectrum-embedding-training
description: Use when you need to build a Spec2Vec spectral similarity score and must
  train (or select a pretrained) Word2Vec embedding from a large processed MS/MS library
  such as the AllPositive GNPS dataset.
when_to_use_negative:
- 'When only a tiny spectral dataset is available: missing words become a relevant
  issue when models are trained on smaller datasets, and the training set must include
  a large fraction of the fragment ions and losses seen in the query data'
- When a training-data-free metric is required (e.g. quick pairwise cosine/modified
  cosine on raw peaks) — Spec2Vec, unlike cosine scores, needs training data to learn
  fragment peak relationships
- 'For GC-MS data: the study only demonstrated performance on LC-MS data'
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: spec2vec
  role: Provides SpectrumDocument conversion and calc_vector to build documents and
    compute spectrum vectors from the trained Word2Vec model
  repo: https://github.com/iomega/spec2vec
- name: gensim
  role: Hosts the Word2Vec algorithm used to train embeddings on all spectrum documents
    of a chosen dataset
- name: matchms
  role: Imports, cleans, and filters spectra (m/z range, minimum peak count, loss
    words) before document conversion
  repo: https://github.com/matchms/matchms
- name: spec2vec_gnps_data_analysis
  role: Notebooks for training/analysis and plotting on the GNPS dataset; contains
    all additional functions to analyse the data and create the presented plots
  repo: https://github.com/iomega/spec2vec_gnps_data_analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:fbaa293ce3326d72908e4f6c7d237b5fb19616ce70e1a3b4b18a33c5f374c694
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - Word2Vec
  - gensim
  - matchms
  - spec2vec
  - spec2vec_gnps_data_analysis
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# word2vec-spectrum-embedding-training

## Summary

Trains a Word2Vec (gensim) model on mass spectra converted to documents of peak/loss words so that related fragments and neutral losses learn similar vector directions, enabling the learned Spec2Vec similarity score. This is the training step underlying Spec2Vec library matching and molecular networking on GNPS-style MS/MS data.

## When to use

Use when you need to build a Spec2Vec spectral similarity score and must train (or select a pretrained) Word2Vec embedding from a large processed MS/MS library such as the AllPositive GNPS dataset.

## When NOT to use

- When only a tiny spectral dataset is available: missing words become a relevant issue when models are trained on smaller datasets, and the training set must include a large fraction of the fragment ions and losses seen in the query data
- When a training-data-free metric is required (e.g. quick pairwise cosine/modified cosine on raw peaks) — Spec2Vec, unlike cosine scores, needs training data to learn fragment peak relationships
- For GC-MS data: the study only demonstrated performance on LC-MS data

## Inputs

- Processed MS/MS spectra (matchms-cleaned, e.g. from GNPS ALL_GNPS.json / AllPositive dataset on Zenodo record 3979010)
- Documents of peak words (m/z encoded to defined decimal precision) and neutral-loss words ('loss@xxx.xx', 5.0-200.0 Da)
- Optional: pretrained Spec2Vec Word2Vec model files from Zenodo (e.g. zenodo record 4173596)

## Outputs

- Trained Word2Vec model (gensim) over peak/loss word vocabulary
- Intensity-weighted spectrum vectors for library and query spectra (Spec2Vec calc_vector)
- Pairwise Spec2Vec similarity scores (cosine between spectrum vectors)

## How to apply

First process spectra with matchms: keep peaks with m/z in [0, 1000], discard spectra with fewer than 10 peaks, and add neutral losses between 5.0 and 200.0 Da as 'loss@xxx.xx' words. Convert each spectrum into a document where every peak is a word encoding its m/z position to a defined decimal precision; for Spec2Vec, cap the number of kept peaks at 0.5 * parent_mass. Train a Word2Vec model with gensim on all documents of the chosen dataset (e.g. the 94,320 AllPositive library spectra). Then compute each spectrum vector as the intensity-weighted sum of its word vectors (Spec2Vec calc_vector) and score pairs via the cosine between spectrum vectors. Instead of training from scratch, pretrained models can be downloaded from Zenodo (e.g. record 4173596 trained on AllPositive, 3978054 on UniqueInchikey). Set a missing-fraction threshold (e.g. <0.05) to avoid returning scores for spectra whose words are largely outside the learned vocabulary.

## Related tools

- **spec2vec** (Provides SpectrumDocument conversion and calc_vector to build documents and compute spectrum vectors from the trained Word2Vec model) — https://github.com/iomega/spec2vec
- **gensim** (Hosts the Word2Vec algorithm used to train embeddings on all spectrum documents of a chosen dataset)
- **matchms** (Imports, cleans, and filters spectra (m/z range, minimum peak count, loss words) before document conversion) — https://github.com/matchms/matchms
- **spec2vec_gnps_data_analysis** (Notebooks for training/analysis and plotting on the GNPS dataset; contains all additional functions to analyse the data and create the presented plots) — https://github.com/iomega/spec2vec_gnps_data_analysis

## Examples

```
from gensim.models import Word2Vec; model = Word2Vec(spectrum_documents, sg=1)  # train on all library spectrum documents, per Huber et al. 2021
```

## Evaluation signals

- Trained-model vocabulary covers ≥97% of peaks/losses in query spectra; spectra exceeding a missing-fraction threshold (e.g. <0.05) should be excluded from scoring
- Spec2Vec scores from the model correlate better with structural (Tanimoto) similarity than cosine-based scores
- Downstream library matching on the AllPositive split reaches up to 88% correctly matched query spectra, outperforming cosine in accuracy and true/false-positive ratio at all thresholds
- Embedding quality should match the published Zenodo reference models (4173596 for AllPositive, 3978054 for UniqueInchikey) when trained on the same cleaned data
- AllPositive dataset size invariants hold after filtering: 95,320 spectra total, 77,092 with InChIKey, 94,121 with InChIKey after matchms metadata cleaning

## Limitations

- Spec2Vec requires training data, unlike cosine scores, so a new model must be trained or a pretrained one reused
- It is important that the training set includes a large fraction of the features (fragment ions and losses) seen in the query data
- Missing words become relevant when models are trained on smaller datasets or when more m/z decimals are included in words
- Some remaining peaks/losses not covered by the model may not be chemically meaningful (e.g. many low-Dalton values)

## Evidence

- [discussion] A Word2Vec model is trained on all documents of a chosen dataset using gensim: "A Word2Vec [22] model is trained on all documents of a chosen dataset using gensim [37]."
- [discussion] Peaks become words encoding m/z position to defined decimal precision: "After processing, spectra are converted to documents. For this, every peak is represented by a word that contains its position up to a defined decimal precision"
- [discussion] Peak cap scales with parent mass for Spec2Vec: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] Spectrum vector as intensity-weighted sum of word vectors: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [readme] README describes Word2Vec adaptation to mass fragments and neutral losses: "Where Word2Vec learns relationships between words in sentences, **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra."
- [discussion] Neutral losses added as loss words between 5.0 and 200.0 Da: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
