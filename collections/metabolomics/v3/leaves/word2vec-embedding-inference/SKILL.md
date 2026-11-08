---
name: word2vec-embedding-inference
description: Use when you need learnt spectral embeddings from a large MS/MS corpus
  (e.g. GNPS-derived UniqueInchikey or AllPositive datasets) instead of peak-matching
  cosine scores, to compare spectra or detect structurally related molecules.
when_to_use_negative:
- When no training data are available — unlike cosine scores, Spec2Vec needs training
  data to learn fragment peak relationships
- When the training set does not include a large fraction of the fragment/loss features
  present in the query data (e.g. small training corpora, or peak words with more
  m/z decimals, causing many missing words)
- When only structural similarity between known molecules is needed — Tanimoto on
  InChIKey-derived structures is the direct measure, not spectral embeddings
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: spec2vec
  role: Implements the Spec2Vec similarity score computed from Word2Vec-derived spectrum
    vectors
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Imports, cleans, and processes spectra into documents before embedding-based
    scoring
  repo: https://github.com/matchms/matchms
- name: gensim
  role: Provides the Word2Vec implementation used to train the spectral embedding
    model
- name: Word2Vec
  role: NLP algorithm adapted to learn co-occurrence relationships between mass fragments
    and neutral losses
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:1672eedd0c41e437168fcb7400f83d6d04b49a31e5ff08a3db497b9511247135
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - Word2Vec
  - gensim
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# word2vec-embedding-inference

## Summary

Adapts Word2Vec (natural language processing) to MS/MS data by training on documents of peak and loss words so that related fragments and neutral losses are embedded as vectors pointing in similar directions, enabling learnt spectral similarity scores such as Spec2Vec.

## When to use

Use when you need learnt spectral embeddings from a large MS/MS corpus (e.g. GNPS-derived UniqueInchikey or AllPositive datasets) instead of peak-matching cosine scores, to compare spectra or detect structurally related molecules.

## When NOT to use

- When no training data are available — unlike cosine scores, Spec2Vec needs training data to learn fragment peak relationships
- When the training set does not include a large fraction of the fragment/loss features present in the query data (e.g. small training corpora, or peak words with more m/z decimals, causing many missing words)
- When only structural similarity between known molecules is needed — Tanimoto on InChIKey-derived structures is the direct measure, not spectral embeddings

## Inputs

- Preprocessed MS/MS spectra (e.g. matchms-cleaned GNPS ALL_GNPS.json derivatives; AllPositive: 95,320 spectra, UniqueInchikey: 12,797 spectra)
- Spectral documents of peak words and 'loss@xxx.xx' loss words (losses 5.0–200.0 Da)
- A trained Word2Vec/gensim model (Zenodo 3978054 for UniqueInchikey; Zenodo 4173596 for AllPositive)

## Outputs

- Word2Vec model embedding fragment/loss words in latent vector space
- Intensity-weighted spectrum vectors (one per spectrum)
- Spec2Vec similarity scores between spectra (e.g. a 12,797 x 12,797 score matrix)

## How to apply

Convert each processed spectrum into a document where every peak is represented by a word encoding its m/z position up to a defined decimal precision, and add neutral losses between 5.0 and 200.0 Da as 'loss@xxx.xx' words. Train a Word2Vec model on all documents of the chosen dataset using gensim (e.g. the pre-trained model for the 12,797-spectrum UniqueInchikey dataset is available at Zenodo record 3978054). Compute each spectrum vector vS as the intensity-weighted sum of its word vectors (vS = Σ wi·vi), then score spectrum pairs via the cosine similarity between the two spectrum vectors. Guard against out-of-model spectra by setting a missing-fraction threshold (e.g. <0.05) so scores are not returned for spectra far outside the learned peaks and losses.

## Related tools

- **spec2vec** (Implements the Spec2Vec similarity score computed from Word2Vec-derived spectrum vectors) — https://github.com/iomega/spec2vec
- **matchms** (Imports, cleans, and processes spectra into documents before embedding-based scoring) — https://github.com/matchms/matchms
- **gensim** (Provides the Word2Vec implementation used to train the spectral embedding model)
- **Word2Vec** (NLP algorithm adapted to learn co-occurrence relationships between mass fragments and neutral losses)

## Examples

```
from spec2vec import Spec2Vec; from gensim.models import Word2Vec; model = Word2Vec.load('spec2vec_Models/uniqueinchikey.model'); spec2vec = Spec2Vec(model, intensity_weighting_power=0.5)
```

## Evaluation signals

- Spectrum vectors must be computable as the weighted sum Σ wi·vi over each spectrum's peak/loss words using the trained model
- Spec2Vec scores should correlate better with structural (Tanimoto) similarity than cosine or modified cosine scores — e.g. for the top 0.1% of ~82M pairs on UniqueInchikey, the Spec2Vec column shows higher average Tanimoto
- Missing-word fraction per spectrum should stay below the chosen threshold (e.g. <0.05); spectra exceeding it should be excluded rather than scored
- Pre-trained model provenance must match the dataset: Zenodo 3978054 (UniqueInchikey) or Zenodo 4173596 (AllPositive)

## Limitations

- Spec2Vec requires training data, unlike cosine scores, and the training set must include a large fraction of the fragment/loss features seen in query data
- Missing words become a relevant issue when models are trained on smaller datasets or when more decimals from peak m/z are included in words
- Other applications such as network analysis may benefit more from a massive training set or retraining the model including the new experimental spectra
- Some remaining peaks/losses not covered by the model may not be chemically meaningful (e.g. many low-Dalton values)

## Evidence

- [intro] Adapting Word2Vec, a well-established machine learning technique in natural language processing: "Adapting Word2Vec, a well-established machine learning technique in natural language processing [22], Spec2Vec learns from co-occurrences across large datasets"
- [discussion] A Word2Vec model is trained on all documents of a chosen dataset using gensim: "A Word2Vec [22] model is trained on all documents of a chosen dataset using gensim [37]."
- [discussion] we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [discussion] neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [readme] **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra.: "Where Word2Vec learns relationships between words in sentences, **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra."
