---
name: gensim-word2vec-model-inference
description: Use when you have a pre-trained Spec2Vec Word2Vec model (e.g. Zenodo
  4173596, AllPositive) and processed peak/loss-word documents, and need spectrum-level
  vectors or pairwise spectral similarity scores.
when_to_use_negative:
- Query spectra far outside the training vocabulary — missing fraction exceeds the
  ~0.05 threshold, so returned scores are unreliable; retrain or skip.
- No training data are available for your compound class; Spec2Vec requires training
  data unlike plain cosine scores, so use cosine/modified cosine instead.
- Data are GC-MS spectra — the approach was only demonstrated on LC-MS data and GC-MS
  utility was not assessed.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: spec2vec
  role: Provides the spectrum-document conversion and Spec2Vec similarity score implementation
    on top of a gensim Word2Vec model
  repo: https://github.com/iomega/spec2vec
- name: matchms
  role: Imports, cleans metadata, and processes spectra before document conversion;
    computes pairwise similarities
  repo: https://github.com/matchms/matchms
- name: gensim
  role: Trains and stores the Word2Vec model from which word vectors for peaks and
    losses are loaded and summed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:2c902d7ba1a4dd0af7c84ac3cfbceb460f1e6a6b21ed30c8794928629dd65691
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - Word2Vec
  - gensim
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# gensim-word2vec-model-inference

## Summary

Convert processed MS/MS spectra into fixed-length embeddings by summing intensity-weighted gensim Word2Vec word vectors for their peak and neutral-loss tokens, then compare spectra via cosine of the embeddings (the Spec2Vec inference stage).

## When to use

Use when you have a pre-trained Spec2Vec Word2Vec model (e.g. Zenodo 4173596, AllPositive) and processed peak/loss-word documents, and need spectrum-level vectors or pairwise spectral similarity scores.

## When NOT to use

- Query spectra far outside the training vocabulary — missing fraction exceeds the ~0.05 threshold, so returned scores are unreliable; retrain or skip.
- No training data are available for your compound class; Spec2Vec requires training data unlike plain cosine scores, so use cosine/modified cosine instead.
- Data are GC-MS spectra — the approach was only demonstrated on LC-MS data and GC-MS utility was not assessed.

## Inputs

- Processed MS/MS spectra (e.g. matchms-cleaned MGF/JSON, such as GNPS ALL_GNPS.json derivatives)
- Trained gensim Word2Vec / Spec2Vec model (Zenodo 4173596 AllPositive or 3978054 UniqueInchikey)
- Peak intensity values used as token weights

## Outputs

- Per-spectrum fixed-length embedding vectors (dimension = Word2Vec model vector size)
- Pairwise Spec2Vec cosine similarity scores (spec2vec_scores.csv)
- Per-spectrum missing-fraction values (spec2vec_scores.csv)

## How to apply

Convert each processed spectrum into a document of peak words (m/z encoded at a fixed decimal precision) and neutral-loss words ('loss@xxx.xx' for losses between 5.0 and 200.0 Da), keeping at most 0.5 x parent_mass peaks per spectrum. Load the trained gensim Word2Vec model (e.g. from zenodo.org/record/4173596 for AllPositive) and compute the spectrum vector vS = sum_i wi*vi, weighting each token's word vector by its peak intensity. For each pair compute g(s1,s2) = cosine(vS1, vS2). Also compute each spectrum's missing fraction, 1 - (covered weighted intensity)/(total weighted intensity), i.e. the intensity fraction of tokens absent from the model vocabulary; flag pairs where it exceeds the recommended <0.05 threshold since scores for spectra far outside the learned peaks/losses are unreliable. Write the pairwise cosine scores and per-spectrum missing fractions to spec2vec_scores.csv.

## Related tools

- **spec2vec** (Provides the spectrum-document conversion and Spec2Vec similarity score implementation on top of a gensim Word2Vec model) — https://github.com/iomega/spec2vec
- **matchms** (Imports, cleans metadata, and processes spectra before document conversion; computes pairwise similarities) — https://github.com/matchms/matchms
- **gensim** (Trains and stores the Word2Vec model from which word vectors for peaks and losses are loaded and summed)

## Examples

```
from gensim.models import Word2Vec; model = Word2Vec.load("spec2vec_AllPositive_model.model"); v = sum(w*v_[i] for i, w in zip(model.wv[word], weights))
```

## Evaluation signals

- Per-spectrum missing fraction is <0.05 for spectra considered in-model; higher values flag out-of-model spectra.
- Number of kept peaks per spectrum never exceeds 0.5 x parent_mass after capping.
- Loss tokens use the 'loss@xxx.xx' format and fall within 5.0–200.0 Da.
- Output spec2vec_scores.csv contains both pairwise cosine scores and per-spectrum missing fractions.
- Spec2Vec scores should correlate better with Tanimoto structural similarity than cosine-based scores on reference pairs.

## Limitations

- Spec2Vec needs training data to learn fragment peak relationships, unlike cosine scores.
- The training set must include a large fraction of the fragment ions and losses seen in query data; missing words become relevant with smaller training sets or more m/z decimals.
- Only demonstrated on LC-MS data; GC-MS utility is untested.
- Some remaining peaks/losses not covered by the model may not be chemically meaningful (e.g. many low-Dalton values).

## Evidence

- [discussion] we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [discussion] For the similarity between two spectra we then compute the cosine score between two spectrum vectors: g(s1,s2)=cosine(vS1,vS2): "For the similarity between two spectra we then compute the cosine score between two spectrum vectors: g(s1,s2)=cosine(vS1,vS2)"
- [discussion] the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided.: "By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided."
- [readme] Where Word2Vec learns relationships between words in sentences, **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra.: "Where Word2Vec learns relationships between words in sentences, **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra."
