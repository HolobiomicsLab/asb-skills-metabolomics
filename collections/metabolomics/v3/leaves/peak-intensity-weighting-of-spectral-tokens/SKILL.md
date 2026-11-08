---
name: peak-intensity-weighting-of-spectral-tokens
description: 'Use when reconstructing Spec2Vec similarity scores from a trained Word2Vec
  model: processed MS/MS spectra must be converted into peak/loss word documents and
  embedded into fixed-length vectors via intensity-weighted summation.'
when_to_use_negative:
- No trained Word2Vec model is available — unlike cosine scores, Spec2Vec requires
  training data to learn fragment peak relationships
- Query spectra are far outside the training vocabulary (missing fraction exceeds
  the <0.05 threshold) — returned scores would be unreliable
- Input is raw, unprocessed spectra without peak filtering (m/z range [0,1000], >=10
  peaks) and metadata cleaning via matchms
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: spec2vec
  role: Python package implementing spectrum documents, intensity-weighted spectrum
    vector computation, and Spec2Vec cosine scoring
  repo: https://github.com/iomega/spec2vec
- name: gensim
  role: Trains/hosts the underlying Word2Vec model from which token word vectors are
    retrieved and summed
- name: matchms
  role: Imports, cleans, and preprocesses MS/MS spectra (metadata correction, peak
    filtering) before document conversion and embedding
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:e6e937d123672107ae5e1a9cc820309bda6302af6ea97c4f46589740cb5e3eaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - gensim
  - matchms
  - spec2vec
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# peak-intensity-weighting-of-spectral-tokens

## Summary

Compute a Spec2Vec spectrum embedding as the peak-intensity-weighted sum of its fragment and neutral-loss word vectors from a trained gensim Word2Vec model, so that spectral similarity reduces to a cosine between two fixed-length vectors.

## When to use

Use when reconstructing Spec2Vec similarity scores from a trained Word2Vec model: processed MS/MS spectra must be converted into peak/loss word documents and embedded into fixed-length vectors via intensity-weighted summation.

## When NOT to use

- No trained Word2Vec model is available — unlike cosine scores, Spec2Vec requires training data to learn fragment peak relationships
- Query spectra are far outside the training vocabulary (missing fraction exceeds the <0.05 threshold) — returned scores would be unreliable
- Input is raw, unprocessed spectra without peak filtering (m/z range [0,1000], >=10 peaks) and metadata cleaning via matchms

## Inputs

- Processed MS/MS spectra (e.g. matchms-cleaned MGF/JSON, peaks with m/z and intensities, precursor parent mass)
- Spectrum documents of peak and loss words (peak words with m/z at fixed decimal precision, 'loss@xxx.xx' for losses 5.0-200.0 Da)
- Trained Spec2Vec Word2Vec model (gensim), e.g. zenodo.org/record/4173596 for AllPositive or zenodo.org/record/3978054 for UniqueInchikey

## Outputs

- Fixed-length spectrum vectors vS (one intensity-weighted embedding per spectrum)
- Pairwise Spec2Vec cosine similarity scores (e.g. spec2vec_scores.csv)
- Per-spectrum missing fractions (fraction of weighted intensity from tokens not in the model vocabulary)

## How to apply

Load a pretrained gensim Word2Vec model (e.g. from the AllPositive deposit at zenodo.org/record/4173596) and convert processed spectra into documents of peak words (m/z position at defined decimal precision) plus neutral losses between 5.0 and 200.0 Da encoded as 'loss@xxx.xx', keeping at most 0.5 x parent_mass peaks per spectrum. For each spectrum, compute the spectrum vector vS = sum_i wi*vi, where each token's word vector vi is weighted by its peak intensity wi. For each spectrum, also compute the missing fraction as 1 - (model-covered weighted intensity)/(total weighted intensity), i.e. the weighted intensity contribution of tokens absent from the model vocabulary; flag spectra with missing fraction >= 0.05 as out-of-model. Finally compute the pairwise Spec2Vec score as the cosine g(s1,s2) = cosine(vS1, vS2) and write the scores and missing fractions (e.g. to spec2vec_scores.csv). The intensity weighting is essential because it makes the embedding reflect which fragments dominate the spectrum rather than treating every peak/loss word equally.

## Related tools

- **spec2vec** (Python package implementing spectrum documents, intensity-weighted spectrum vector computation, and Spec2Vec cosine scoring) — https://github.com/iomega/spec2vec
- **gensim** (Trains/hosts the underlying Word2Vec model from which token word vectors are retrieved and summed)
- **matchms** (Imports, cleans, and preprocesses MS/MS spectra (metadata correction, peak filtering) before document conversion and embedding) — https://github.com/matchms/matchms

## Examples

```
from spec2vec import Spec2Vec; spec2vec_similarity = Spec2Vec(model=word2vec_model, intensity_weighting_power=0.5, allowed_missing_percentage=5.0); scores = spec2vec_similarity.matrix(documents, documents)
```

## Evaluation signals

- Each computed vS has the same fixed length as the Word2Vec model's vector_size (low-dimensional embedding)
- Missing fraction is in [0, 1]; spectra with missing fraction < 0.05 are considered in-model and scoreable
- Identical spectra yield cosine score 1.0; all pairwise scores lie within [-1, 1]
- Spot-check that vS equals the manual sum of wi*vi for a small spectrum's peak/loss tokens (intensity weighting actually applied)
- Output CSV contains one Spec2Vec cosine score per spectrum pair plus a per-spectrum missing-fraction column

## Limitations

- Spec2Vec needs training data, and the training set must include a large fraction of the fragment ions and losses seen in the query data
- Missing words become a relevant issue when models are trained on smaller datasets or when more decimals from peak m/z are included in words
- Some peaks/losses not covered by the model may not be chemically meaningful (e.g. many low-Dalton values)
- Performance only demonstrated on LC-MS data; GC-MS utility not assessed

## Evidence

- [discussion] we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [discussion] cosine between spectrum vectors: "For the similarity between two spectra we then compute the cosine score between two spectrum vectors: g(s1,s2)=cosine(vS1,vS2)"
- [discussion] missing fraction threshold: "By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided."
- [readme] Word2Vec adapted to fragments and losses: "Where Word2Vec learns relationships between words in sentences, **spec2vec** does so for mass fragments and neutral losses in MS/MS spectra."
