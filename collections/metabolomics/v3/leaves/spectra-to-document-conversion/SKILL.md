---
name: spectra-to-document-conversion
description: Use when preparing preprocessed MS/MS spectra (e.g. cleaned MGF/GNPS
  library spectra) as input for training a Word2Vec/Spec2Vec model or for scoring
  spectra with a pre-trained Spec2Vec model.
when_to_use_negative:
- Computing cosine or modified cosine scores only — these operate directly on peak
  lists and need no document conversion
- 'Spectra far outside the training data of the available model: missing words dominate
  and Spec2Vec scores become unreliable (use the missing-fraction threshold, e.g.
  <0.05, to detect this)'
- Datasets too small for meaningful Word2Vec training — missing words become a relevant
  issue when models are trained on smaller datasets or with more m/z decimals
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: spec2vec
  role: Provides Spec2VecDocument / document conversion of spectra into peak and loss
    word documents for Word2Vec training and scoring
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Upstream filtering and metadata cleaning of spectra before document conversion;
    also supplies the cosine/modified cosine scores compared against Spec2Vec
  repo: https://github.com/matchms/matchms
- name: Word2Vec (gensim)
  role: Trains embeddings on the converted peak/loss word documents
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:ee92f971cb7731a5700cc4a431489fa000b7ea745ac7aef29d0d205856cf7e66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - Word2Vec (gensim)
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectra-to-document-conversion

## Summary

Converts processed MS/MS spectra into 'documents' of peak and loss words so that Word2Vec-style models (Spec2Vec) can learn embeddings of fragment/loss relationships. It is the representation step that precedes training a Spec2Vec model or computing Spec2Vec similarity scores.

## When to use

Use when preparing preprocessed MS/MS spectra (e.g. cleaned MGF/GNPS library spectra) as input for training a Word2Vec/Spec2Vec model or for scoring spectra with a pre-trained Spec2Vec model.

## When NOT to use

- Computing cosine or modified cosine scores only — these operate directly on peak lists and need no document conversion
- Spectra far outside the training data of the available model: missing words dominate and Spec2Vec scores become unreliable (use the missing-fraction threshold, e.g. <0.05, to detect this)
- Datasets too small for meaningful Word2Vec training — missing words become a relevant issue when models are trained on smaller datasets or with more m/z decimals

## Inputs

- Preprocessed MS/MS spectra (matchms Spectrum objects or MGF files from GNPS library / UniqueInchikey dataset)
- Peak list with m/z and intensity values per spectrum
- Precursor/parent mass metadata per spectrum
- Pre-trained Word2Vec/Spec2Vec model (e.g. Zenodo 3978054 for UniqueInchikey) when scoring rather than training

## Outputs

- One document of peak words and 'loss@xxx.xx' loss words per spectrum
- Document corpus suitable for gensim Word2Vec training or Spec2Vec scoring
- Spectrum vectors (intensity-weighted sums of word vectors) when combined with a trained model

## How to apply

After filtering spectra in matchms (peaks within m/z [0, 1000], at least 10 peaks per spectrum, intensity filter applied where relevant), convert each spectrum into a document in which every peak becomes a word encoding its position up to a defined decimal precision. Additionally, add neutral losses between 5.0 and 200.0 Da as 'loss@xxx.xx' words alongside the peak words. Cap the number of kept peaks per spectrum at 0.5 x parent mass, and use a fixed number of decimals for peak m/z so that identical fragments map to the same word across spectra. The resulting documents are the corpus on which a gensim Word2Vec model is trained, or against which pre-trained models (e.g. Zenodo record 3978054, trained on the UniqueInchikey dataset) are applied to compute spectrum vectors and Spec2Vec similarities.

## Related tools

- **spec2vec** (Provides Spec2VecDocument / document conversion of spectra into peak and loss word documents for Word2Vec training and scoring) — https://github.com/iomega/spec2vec
- **matchms** (Upstream filtering and metadata cleaning of spectra before document conversion; also supplies the cosine/modified cosine scores compared against Spec2Vec) — https://github.com/matchms/matchms
- **Word2Vec (gensim)** (Trains embeddings on the converted peak/loss word documents)

## Examples

```
from spec2vec import SpectrumDocument; documents = [SpectrumDocument(s, n_decimals=2) for s in spectra]
```

## Evaluation signals

- Every document contains peak words encoding m/z positions at a defined decimal precision plus 'loss@xxx.xx' words for losses between 5.0 and 200.0 Da
- Spectra with fewer than 10 fragment peaks and peaks with m/z outside [0, 1000] have been removed before conversion
- Missing-fraction check: fraction of document words absent from the trained model vocabulary is below the chosen threshold (e.g. <0.05)
- Downstream Spec2Vec similarity matrix (e.g. 12,797 x 12,797 on UniqueInchikey) can be computed and shows the expected correlation pattern with Tanimoto similarity versus cosine scores
- Number of kept peaks per spectrum respects the max(npeaks) = 0.5 x parent_mass cap

## Limitations

- Spec2Vec needs training data to learn fragment peak relationships, unlike cosine scores; the training set must include a large fraction of the fragment/loss features seen in query data
- Missing words become a relevant issue when models are trained on smaller datasets or when more decimals from peak m/z are included in words
- Some remaining peaks/losses not covered by the model (about 3% in the study) may not be chemically meaningful (e.g. many low-Dalton values)

## Evidence

- [discussion] After processing, spectra are converted to documents. For this, every peak is represented by a word that contains its position up to a defined decimal precision: "After processing, spectra are converted to documents. For this, every peak is represented by a word that contains its position up to a defined decimal precision"
- [discussion] Neutral losses added as loss words: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] Peak count cap scales with parent mass: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] Spectrum vector is intensity-weighted sum of word vectors: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [readme] README: spec2vec learns relationships for mass fragments and neutral losses: "Where Word2Vec learns relationships between words in sentences, **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra."
