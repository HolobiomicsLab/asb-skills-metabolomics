---
name: uncovered-peak-missing-fraction-computation
description: Use when computing Spec2Vec similarity scores for spectra against a pretrained
  Word2Vec model and you must decide whether the score is trustworthy because query
  peaks/losses fall outside the model vocabulary.
when_to_use_negative:
- Scoring with plain cosine or modified cosine similarity — these do not depend on
  a learned vocabulary, so missing fractions are irrelevant
- The model was trained on a small dataset or with high m/z decimal precision, where
  most query words may be missing and the <0.05 threshold would flag nearly everything
  — retraining including the new experimental spectra may be needed
- The training set does not include a large fraction of the fragment-ion and neutral-loss
  features seen in the query data (e.g. far-out-of-model compound classes)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Spec2Vec
  role: Provides the spectral embedding machinery and missing-fraction accounting
    for out-of-model words
  repo: https://github.com/iomega/spec2vec
- name: Word2Vec
  role: Underlying model whose vocabulary coverage determines which peak/loss words
    are counted as missing
  repo: https://github.com/iomega/spec2vec
- name: gensim
  role: Framework in which the Spec2Vec Word2Vec model is trained and loaded for token
    lookups
- name: matchms
  role: Imports, cleans, and processes the MS/MS spectra that are converted to word
    documents prior to missing-fraction computation
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008724
    title: 'Spec2Vec: Improved mass spectral similarity scoring through learning of
      structural relationships'
  source_content_hash: sha256:777b5e48890cae1f1b8c4ab39e47791de9b1c789358d97ab7309e0d27d113ae1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/spec2vec
  tools:
  - Spec2Vec
  - Word2Vec
  - gensim
  - matchms
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# uncovered-peak-missing-fraction-computation

## Summary

Quantifies how much of a spectrum's intensity-weighted peak/loss signal is represented by tokens absent from a trained Spec2Vec (Word2Vec/gensim) model vocabulary, so that Spec2Vec cosine scores are only returned for spectra actually 'in-model'.

## When to use

Use when computing Spec2Vec similarity scores for spectra against a pretrained Word2Vec model and you must decide whether the score is trustworthy because query peaks/losses fall outside the model vocabulary.

## When NOT to use

- Scoring with plain cosine or modified cosine similarity — these do not depend on a learned vocabulary, so missing fractions are irrelevant
- The model was trained on a small dataset or with high m/z decimal precision, where most query words may be missing and the <0.05 threshold would flag nearly everything — retraining including the new experimental spectra may be needed
- The training set does not include a large fraction of the fragment-ion and neutral-loss features seen in the query data (e.g. far-out-of-model compound classes)

## Inputs

- Processed MS/MS spectra converted to documents of peak and loss words (peak words encoding m/z at defined decimal precision, losses 5.0–200.0 Da)
- Trained Spec2Vec Word2Vec gensim model (e.g. zenodo.org/record/4173596, AllPositive dataset)
- Peak intensity weights per spectrum (wi) and parent mass values used for the 0.5 x parent_mass peak cap

## Outputs

- Per-spectrum missing fraction (1 - covered weighted intensity / total weighted intensity)
- Flagged set of spectra/pairs exceeding the missing-fraction threshold (<0.05 considered in-model)
- Pairwise Spec2Vec cosine scores with per-spectrum missing fractions, e.g. written to spec2vec_scores.csv

## How to apply

After converting each processed spectrum into a document of peak words and neutral-loss words ('loss@xxx.xx', losses between 5.0 and 200.0 Da, at most 0.5 x parent_mass peaks kept), compute the spectrum vector vS = sum_i wi*vi as the intensity-weighted sum of word vectors. For each token, check whether its word exists in the loaded model vocabulary (models e.g. from zenodo.org/record/4173596 trained on AllPositive); the missing fraction is then 1 - (sum of model-covered weighted intensity) / (sum of total weighted intensity). Apply the recommended threshold of missing fraction <0.05: pairs where either spectrum exceeds it should be flagged or excluded rather than scored, since vectors far outside the learned peaks/losses give unreliable cosine similarities. Missing words become a real issue for models trained on smaller datasets or when more decimals of peak m/z are included in words. Write both the pairwise Spec2Vec cosine scores g(s1,s2) = cosine(vS1,vS2) and the per-spectrum missing fractions (e.g. to spec2vec_scores.csv) so downstream users can filter.

## Related tools

- **Spec2Vec** (Provides the spectral embedding machinery and missing-fraction accounting for out-of-model words) — https://github.com/iomega/spec2vec
- **Word2Vec** (Underlying model whose vocabulary coverage determines which peak/loss words are counted as missing) — https://github.com/iomega/spec2vec
- **gensim** (Framework in which the Spec2Vec Word2Vec model is trained and loaded for token lookups)
- **matchms** (Imports, cleans, and processes the MS/MS spectra that are converted to word documents prior to missing-fraction computation) — https://github.com/matchms/matchms

## Examples

```
model = gensim.models.Word2Vec.load('spec2vec_model_allpositive.model'); missing_fraction = 1 - covered_weighted_intensity / total_weighted_intensity
```

## Evaluation signals

- Missing fraction is bounded in [0, 1]; a spectrum whose every token is in-vocabulary yields exactly 0
- The missing fraction equals the total weighted intensity of tokens absent from the model vocabulary divided by total weighted intensity — recompute on a small subset and diff against the pipeline output
- Spectra/pairs flagged with missing fraction >= 0.05 are excluded from returned Spec2Vec scores
- Per-spectrum missing fractions are emitted alongside the pairwise Spec2Vec cosine scores (e.g. in spec2vec_scores.csv), not only the scores themselves

## Limitations

- Missing words mostly become relevant when Spec2Vec models are trained on smaller datasets or when more decimals from peak m/z are included in words
- Not all remaining uncovered peaks/losses (e.g. many low-Dalton values) may be chemically meaningful
- Spec2Vec requires training data, unlike cosine scores, and the training set must include a large fraction of the fragment ions and losses seen in the query data
- Returning Spec2Vec scores for spectra far outside the learned peaks (and losses) should be avoided via the missing-fraction threshold

## Evidence

- [discussion] missing fraction: "the maximum number of kept peaks per spectrum was set to scale linearly with the estimated parent mass: max(npeaks)=0.5·parentmass"
- [discussion] missing fraction threshold: "By setting a threshold for the missing fraction (e.g. <0.05), returning Spec2Vec similarity scores for spectra far outside the learned peaks (and losses) can be avoided."
- [discussion] missing words issue: "missing words will mostly become a relevant issue when Spec2Vec models are trained on smaller datasets, or when more decimals from peak m/z are included in words."
- [discussion] loss words: "In addition to all peaks of a spectrum, neutral losses between 5.0 and 200.0 Da were added as “loss@xxx.xx”."
- [discussion] training data requirement: "for Spec2Vec to work it is important that the training set includes a large fraction of the features (fragment ions and losses) that we see in the data"
