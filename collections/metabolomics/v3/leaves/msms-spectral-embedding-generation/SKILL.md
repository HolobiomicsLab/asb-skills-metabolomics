---
name: msms-spectral-embedding-generation
description: Use when HR-MS/MS spectra (e.g., MGF-format annotated spectra from MSnLib
  or in-house libraries) must be converted into MS2DeepScore feature vectors before
  training or applying spectra classifiers.
when_to_use_negative:
- The training dataset is highly imbalanced — the article states classification into
  substance classes will most often fail in that case and AnnoMe cannot resolve it
- Embeddings have already been computed and cached for the same spectra — regenerate
  only for new/changed spectra, not as a routine re-run
- The task is spectral filtering or classification itself rather than feature preparation
  — use the AnnoMe filtering GUI or classification GUI instead
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Python package whose workflow computes and caches MS2DeepScore embeddings
    for the training and inference spectra
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Structure-informed embedding model applied to HR-MS/MS spectra to produce
    the feature vectors used by the classifiers
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Python toolkit used to install and run the AnnoMe package (e.g., `uv run`
    commands) in which embedding generation executes
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:21987a5f2dad6023779daa95d9b2f21de891e6ed6d91ec928e320010f0604fab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# msms-spectral-embedding-generation

## Summary

Compute structure-informed MS2DeepScore embeddings for HR-MS/MS spectra (e.g., filtered annotated MSnLib spectra) so that spectra can be represented as fixed-length vectors for downstream training of 'relevant' vs 'other' classifiers within the AnnoMe workflow.

## When to use

Use when HR-MS/MS spectra (e.g., MGF-format annotated spectra from MSnLib or in-house libraries) must be converted into MS2DeepScore feature vectors before training or applying spectra classifiers.

## When NOT to use

- The training dataset is highly imbalanced — the article states classification into substance classes will most often fail in that case and AnnoMe cannot resolve it
- Embeddings have already been computed and cached for the same spectra — regenerate only for new/changed spectra, not as a routine re-run
- The task is spectral filtering or classification itself rather than feature preparation — use the AnnoMe filtering GUI or classification GUI instead

## Inputs

- Annotated and labeled HR-MS/MS spectra in MGF format (e.g., filtered MSnLib library spectra, in-house BOKU University database spectra, or experimental spectra from wheat ear / Paulownia tomentosa / Glycyrrhizza uralensis extracts)
- AnnoMe package environment installed via uv (with downloaded public MS/MS resources, approximately 11GB)

## Outputs

- Per-spectrum MS2DeepScore embedding vectors cached on disk, keyed to the input spectra and reusable for classifier training and inference

## How to apply

Load the annotated, labeled MGF spectra (e.g., the 30703 filtered MSnLib spectra of prenylated (iso-)flavonoids and other compounds) into the AnnoMe environment set up with uv. Compute MS2DeepScore embeddings for every spectrum; this is the limiting step, taking about 4 hours on a standard laptop (Intel Core Ultra 5 125U, 12 cores, 16GB RAM, SSD), and it should be scheduled to run once over night or a weekend. Embeddings are cached on disk after the first computation, which reduces subsequent runs from hours to a few minutes, so reuse the cache for classifier training, ensemble evaluation, and negative-control inference rather than recomputing. The resulting per-spectrum embedding vectors serve as the input features for training LDA, neural-network, and SVM classifiers with cross-validation and different random seeds, whose majority vote yields the final 'relevant'/'other' assignment.

## Related tools

- **AnnoMe** (Python package whose workflow computes and caches MS2DeepScore embeddings for the training and inference spectra) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Structure-informed embedding model applied to HR-MS/MS spectra to produce the feature vectors used by the classifiers) — https://github.com/chrboku/AnnoMe
- **uv** (Python toolkit used to install and run the AnnoMe package (e.g., `uv run` commands) in which embedding generation executes) — https://github.com/astral-sh/uv

## Examples

```
uv run annome_downloadresources && uv run annome_classificationgui
```

## Evaluation signals

- Every spectrum in the input MGF file set has a corresponding embedding vector available (no missing rows) before classifier training begins
- Embeddings are cached on disk after the first run; a second run over the same spectra completes in minutes rather than hours
- Downstream classifiers (LDA, NN, SVM) trained on the embeddings reproduce the reported performance: 64 prenylated 'relevant' test spectra plus 7312 wheat ear extract spectra scored per Table 1
- The ground-truth negative control of 155 non-prenylated (iso-)flavonoid spectra classified from these embeddings yields ~58% correct 'other' assignments in negative mode and ~67% in positive mode (Table ST2)
- Embedding generation resources match expectations: ~11GB downloaded resources and roughly 4 hours runtime on a 12-core, 16GB laptop for the limiting step

## Limitations

- Embedding generation is the most time-consuming step (~4 hours on a standard laptop; 12 cores, 16GB RAM) and could not be completed on a less powerful passively-cooled MacBook Air with 16GB RAM
- Embeddings must be computed once and cached; without the cache, repeated classification runs pay the full hours-long cost
- Even with good embeddings, spectra classification into substance classes is non-trivial and often unsuccessful, especially with highly imbalanced training datasets — users should exercise caution with the results

## Evidence

- [methods] The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop"
- [readme] Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [abstract] By leveraging a user-curated dataset of "relevant" and "other" reference HR-MS/MS spectra: "By leveraging a user-curated dataset of "relevant" and "other" reference HR-MS/MS spectra"
- [readme] different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
