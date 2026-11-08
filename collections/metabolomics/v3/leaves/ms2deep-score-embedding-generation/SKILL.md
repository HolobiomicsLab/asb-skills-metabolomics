---
name: ms2deep-score-embedding-generation
description: Use when HR-MS/MS spectra (e.g., MGF inference spectra of Glycyrrhiza
  uralensis root or Paulownia tomentosa fruit extracts) need MS2DeepScore structure-informed
  embeddings as classifier input in AnnoMe.
when_to_use_negative:
- An on-disk embedding cache already exists for the same spectra — regeneration takes
  hours, so reuse the cached embeddings and only recompute when inputs change.
- The goal is filtering MGF files for interesting structures rather than producing
  classifier-ready features — use the AnnoMe filtering GUI step instead.
- Training data are highly imbalanced and you expect embeddings alone to fix poor
  classification — the article states the package cannot resolve this failure mode.
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Runs the embedding generation step for inference spectra loaded in the classification
    GUI and manages the on-disk embedding cache
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Supplies the structure-informed spectral embedding model applied to each HR-MS/MS
    spectrum
- name: uv
  role: Launcher for the AnnoMe commands/GUIs that trigger embedding computation (e.g.,
    uv run annome_classificationgui)
  repo: github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:d89e7552771ccdf19296cb624142973e26eb78e4af6c8131d7d7723148022c00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# ms2deep-score-embedding-generation

## Summary

Generates MS2DeepScore structure-informed embeddings for HR-MS/MS spectra inside the AnnoMe package; these embeddings are the feature representation consumed by custom-trained classifiers (LDA, NN, SVM, etc.) for 'relevant'/'other' spectral classification. It is the most time-consuming workflow step, so embeddings are cached on disk after the first computation.

## When to use

Use when HR-MS/MS spectra (e.g., MGF inference spectra of Glycyrrhiza uralensis root or Paulownia tomentosa fruit extracts) need MS2DeepScore structure-informed embeddings as classifier input in AnnoMe.

## When NOT to use

- An on-disk embedding cache already exists for the same spectra — regeneration takes hours, so reuse the cached embeddings and only recompute when inputs change.
- The goal is filtering MGF files for interesting structures rather than producing classifier-ready features — use the AnnoMe filtering GUI step instead.
- Training data are highly imbalanced and you expect embeddings alone to fix poor classification — the article states the package cannot resolve this failure mode.

## Inputs

- MGF file of HR-MS/MS inference spectra (e.g., deposited spectra of Glycyrrhiza uralensis root and Paulownia tomentosa fruit extracts)
- Reference HR-MS/MS spectra labeled 'relevant'/'other' (e.g., MSnLib libraries, BOKU University in-house database) that also require embeddings during classifier training

## Outputs

- Per-spectrum MS2DeepScore structure-informed embedding vectors for all HR-MS/MS spectra in the input MGF file
- On-disk embedding cache that lets subsequent AnnoMe classification runs skip recomputation

## How to apply

Load the deposited MGF inference spectra (e.g., Glycyrrhiza uralensis root or Paulownia tomentosa fruit extracts) into AnnoMe via the classification GUI (`uv run annome_classificationgui`), which triggers MS2DeepScore embedding computation for every HR-MS/MS spectrum. This step is compute-bound: it took about 4 hours on a standard laptop (Intel Core Ultra 5 125U, 12 cores, 16GB RAM, SSD, Windows 11) and could not be completed on a less powerful, passively cooled MacBook Air (M4, 16GB RAM), so run it once on sufficient hardware, e.g., overnight or over a weekend. Embeddings are cached on disk after the first run, reducing subsequent runs from hours to a few minutes — never recompute when a cache exists for the same spectra. The rationale is that MS2DeepScore yields structure-informed spectral embeddings so the trained classifiers (LDA, NN, SVM, etc.) operate in a feature space where 'relevant' vs 'other' separation is learnable, and the resulting per-spectrum vectors feed the majority-vote aggregation that produces final classifications.

## Related tools

- **AnnoMe** (Runs the embedding generation step for inference spectra loaded in the classification GUI and manages the on-disk embedding cache) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Supplies the structure-informed spectral embedding model applied to each HR-MS/MS spectrum)
- **uv** (Launcher for the AnnoMe commands/GUIs that trigger embedding computation (e.g., uv run annome_classificationgui)) — github.com/astral-sh/uv

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- A second run over the same spectra completes in a few minutes instead of hours, confirming the on-disk embedding cache was written and is being reused.
- Every HR-MS/MS spectrum in the input MGF file yields an embedding vector so trained classifiers (LDA, NN, SVM) can score all spectra and majority-vote labels can be derived.
- Downstream per-sample 'relevant' counts fall near the reported values: Glycyrrhiza uralensis 231 (25%) and 305 (15%) spectra; Paulownia tomentosa 171–1005 spectra (17%–34%) across samples (Section 3.3 / Table ST4).
- Embedding generation completes on sufficiently powerful hardware (about 4 hours on a 12-core, 16GB-RAM laptop); stalls or failure on weaker machines (e.g., passively cooled MacBook Air) indicate a resource problem rather than a data problem.

## Limitations

- Embedding generation is the most time-consuming step (about 4 hours on a standard laptop with Intel Core Ultra 5 125U, 12 cores, 16GB RAM) and could not be processed on a less powerful MacBook Air (M4, 16GB RAM, passive cooling).
- Embeddings must be generated once (e.g., overnight or a weekend) and rely on disk caching; recomputation is expensive.
- Well-formed embeddings do not guarantee correct classification: classifying MS/MS spectra into substance classes often fails, especially with highly imbalanced training datasets, and the package cannot resolve this.

## Evidence

- [abstract] abstract: "alongside structure-informed embeddings (MS2DeepScore), the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote"
- [methods] methods: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [readme] readme: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop (Intel Core Ultra 5 125U, 12 cores; 16GB main memory; SSD; Windows 11)."
- [discussion] discussion: "especially when the training dataset is highly imbalanced. The presented python package is not capable of resolving this issue."
