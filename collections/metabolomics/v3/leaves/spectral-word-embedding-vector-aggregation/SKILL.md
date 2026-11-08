---
name: spectral-word-embedding-vector-aggregation
description: Use when you have a trained Spec2Vec Word2Vec model and processed spectra
  as peak/loss word documents, and need pairwise spectral similarity scores for library
  matching or molecular networking.
when_to_use_negative:
- No trained model is available and the training set does not include a large fraction
  of the fragment ions and losses present in the query data — Spec2Vec needs training
  data, unlike cosine scores
- The task is plain peak-by-peak cosine matching of already-aligned peaks — use the
  cosine score instead
- Query spectra are far outside the model's learned peaks/losses (missing fraction
  > 0.05), e.g. spectra from an instrument or chemistry domain absent from training
edam_operation: http://edamontology.org/operation_4009
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: spec2vec
  role: Computes spectrum vectors as intensity-weighted sums of Word2Vec embeddings
    and cosine similarity scores between them
  repo: https://github.com/iomega/spec2vec
- name: gensim
  role: Provides the Word2Vec model in which each peak/loss word has an embedding
    vector to aggregate
- name: matchms
  role: Imports, cleans, and processes the spectra into documents of peak and loss
    words prior to vector aggregation
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:94f9daa575e58f899a2a6557c40526bf74a0d2d94fbe47df26dfcab410e981d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - gensim
  - matchms
  - spec2vec
  edam_operation: http://edamontology.org/operation_4009
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectral-word-embedding-vector-aggregation

## Summary

Converts processed MS/MS spectra into fixed-length spectrum vectors by summing each peak/loss word's Word2Vec embedding weighted by peak intensity, then scores spectral similarity as the cosine between two spectrum vectors (the Spec2Vec score).

## When to use

Use when you have a trained Spec2Vec Word2Vec model and processed spectra as peak/loss word documents, and need pairwise spectral similarity scores for library matching or molecular networking.

## When NOT to use

- No trained model is available and the training set does not include a large fraction of the fragment ions and losses present in the query data — Spec2Vec needs training data, unlike cosine scores
- The task is plain peak-by-peak cosine matching of already-aligned peaks — use the cosine score instead
- Query spectra are far outside the model's learned peaks/losses (missing fraction > 0.05), e.g. spectra from an instrument or chemistry domain absent from training

## Inputs

- Trained Spec2Vec Word2Vec model (gensim format, e.g. from Zenodo 4173596 trained on AllPositive)
- Processed MS/MS spectra (matchms-processed, e.g. from GNPS ALL_GNPS.json library) with parent mass and peak intensities
- Peak/loss word documents with peak words encoding m/z position at defined decimal precision and neutral losses between 5.0 and 200.0 Da as 'loss@xxx.xx'

## Outputs

- Pairwise Spec2Vec cosine similarity scores between spectrum vectors (e.g. spec2vec_scores.csv)
- Per-spectrum missing fractions (weighted intensity of tokens absent from model vocabulary)

## How to apply

Load a pretrained gensim Word2Vec model (e.g. the AllPositive model from zenodo.org/record/4173596) and convert each processed spectrum into a document of peak words (m/z position at defined decimal precision) plus neutral losses between 5.0 and 200.0 Da encoded as 'loss@xxx.xx', keeping at most 0.5 x parent_mass peaks per spectrum. For each spectrum compute the spectrum vector vS = sum_i wi*vi, weighting each token's word vector vi by its peak intensity wi. For each spectrum pair compute the Spec2Vec score g(s1,s2) = cosine(vS1, vS2). Also compute a per-spectrum missing fraction as 1 - (sum of model-covered weighted intensity)/(total weighted intensity), and flag pairs where the missing fraction exceeds 0.05 so scores for spectra far outside the learned peaks/losses are not trusted. The rationale is that Word2Vec embeddings place related fragments/losses in similar vector directions, so cosine of intensity-weighted embeddings better tracks structural (Tanimoto) similarity than cosine peak matching.

## Related tools

- **spec2vec** (Computes spectrum vectors as intensity-weighted sums of Word2Vec embeddings and cosine similarity scores between them) — https://github.com/iomega/spec2vec
- **gensim** (Provides the Word2Vec model in which each peak/loss word has an embedding vector to aggregate)
- **matchms** (Imports, cleans, and processes the spectra into documents of peak and loss words prior to vector aggregation) — https://github.com/matchms/matchms

## Examples

```
from spec2vec import Spec2Vec; from matchms.importing import load_from_mgf; scores = calculate_scores(spectra, spectra, Spec2Vec(model=word2vec_model, intensity_weighting_power=0.5), is_symmetric=True)
```

## Evaluation signals

- Per-spectrum missing fraction is below the 0.05 threshold for all spectra whose scores are reported
- Number of peaks kept per spectrum does not exceed 0.5 x parent_mass
- Spec2Vec scores fall in [0, 1] (cosine of non-negative weighted embeddings) and correlate better with Tanimoto structural similarity than cosine-based scores
- Output file (e.g. spec2vec_scores.csv) contains one score per spectrum pair plus per-spectrum missing fractions
- Spectrum vectors have fixed dimensionality equal to the Word2Vec model's embedding size

## Limitations

- Spec2Vec requires training data to learn fragment peak relationships, unlike cosine scores
- For Spec2Vec to work it is important that the training set includes a large fraction of the features (fragment ions and losses) seen in the query data; missing words become relevant for smaller training sets or when more m/z decimals are included
- Network analysis on new experimental spectra may require a massive training set or retraining the model including those spectra
- Performance has only been demonstrated on LC-MS data; GC-MS utility is unassessed

## Evidence

- [discussion] we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi: "we calculate a spectrum vector vS as a weighted sum: vS=∑i=1nwi·vi"
- [discussion] For the similarity between two spectra we then compute the cosine score between two spectrum vectors: "For the similarity between two spectra we then compute the cosine score between two spectrum vectors: g(s1,s2)=cosine(vS1,vS2)"
- [discussion] the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”.: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided.: "By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided."
- [discussion] one limitation of Spec2Vec as compared to cosine scores is that it needs training data to learn the fragment peak relationships: "one limitation of Spec2Vec as compared to cosine scores is that it needs training data to learn the fragment peak relationships"
