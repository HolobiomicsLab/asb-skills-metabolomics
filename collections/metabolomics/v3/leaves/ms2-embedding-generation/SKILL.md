---
name: ms2-embedding-generation
description: Use when labeled 'relevant' and 'other' MS/MS spectra (e.g. MSnLib libraries,
  in-house BOKU database, wheat ear extract) must be converted into MS2DeepScore embeddings
  as classifier input features.
when_to_use_negative:
- When embeddings have already been computed and cached on disk — re-running the generation
  step from scratch wastes hours; reuse the cached embeddings instead.
- When the training dataset is highly imbalanced between 'relevant' and 'other' classes
  — the article states MS/MS classification will most often not be successful and
  the package cannot resolve this issue.
- On resource-constrained hardware without active cooling or sufficient cores/memory
  — the same dataset could not be processed on a less powerful MacBook Air (M4, 16
  GB, passive cooling).
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: AnnoMe
  role: Python package that orchestrates the generation of MS2DeepScore embeddings
    from the user's labeled MS/MS spectra, caching them on disk for subsequent classifier
    training.
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: 'The embedding model itself: produces structure-informed representations of
    MS/MS spectra used as classifier input features.'
- name: uv
  role: Python toolkit used to install and run the AnnoMe package (uv run …) on the
    host machine.
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:d96f30c471c834e7c383da88ecf8f14477127511515ef3e45c8858c06cc666a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 21
---

# ms2-embedding-generation

## Summary

Generates structure-informed MS2DeepScore embeddings for labeled HR-MS/MS training spectra so downstream classifiers (LDA, NN, SVM) can discriminate 'relevant' from 'other' compound classes in AnnoMe.

## When to use

Use when labeled 'relevant' and 'other' MS/MS spectra (e.g. MSnLib libraries, in-house BOKU database, wheat ear extract) must be converted into MS2DeepScore embeddings as classifier input features.

## When NOT to use

- When embeddings have already been computed and cached on disk — re-running the generation step from scratch wastes hours; reuse the cached embeddings instead.
- When the training dataset is highly imbalanced between 'relevant' and 'other' classes — the article states MS/MS classification will most often not be successful and the package cannot resolve this issue.
- On resource-constrained hardware without active cooling or sufficient cores/memory — the same dataset could not be processed on a less powerful MacBook Air (M4, 16 GB, passive cooling).

## Inputs

- Annotated and labeled MS/MS spectra of compounds of interest ('relevant') and others ('other'), e.g. MSnLib spectra libraries and an in-house database obtained at BOKU University
- MS/MS spectra from a wheat ear extract obtained at BOKU University (used as 'other' class)
- Inference dataset spectra from plant extracts known to produce prenylated flavonoids (Paulownia tomentosa fruits; Glycyrrhizza uralensis roots)

## Outputs

- MS2DeepScore structure-informed embedding vectors per MS/MS spectrum (cached on disk after first run)
- Embedded training features consumed by the LDA/NN/SVM classifiers within AnnoMe

## How to apply

Compute MS2DeepScore structure-informed embeddings for the curated labeled MS/MS spectra using the AnnoMe Python package, which wraps MS2DeepScore. This is the limiting computational step: on a standard laptop (Intel Core Ultra 5 125U, 12 cores, 16 GB RAM, SSD, Windows 11) it took 4 hours for the demonstration dataset and failed on a passively-cooled MacBook Air (M4, 16 GB). Embeddings are cached on disk after the first run, so schedule the initial generation overnight or over a weekend and reuse the cache thereafter, which reduces the step from hours to a few minutes. The resulting per-spectrum embedding vectors then serve as the feature space in which polarity- and collision-energy-stratified classifiers are trained with cross-validation and majority-vote aggregation.

## Related tools

- **AnnoMe** (Python package that orchestrates the generation of MS2DeepScore embeddings from the user's labeled MS/MS spectra, caching them on disk for subsequent classifier training.) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (The embedding model itself: produces structure-informed representations of MS/MS spectra used as classifier input features.)
- **uv** (Python toolkit used to install and run the AnnoMe package (uv run …) on the host machine.) — https://github.com/astral-sh/uv

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- Embedding generation completes for the full labeled spectrum set and files are written to the disk cache, so subsequent runs skip recomputation.
- On re-execution after caching, the embedding step's duration drops from hours to a few minutes rather than repeating the 4-hour initial run.
- The resulting embeddings feed classifier training whose cross-validated specificity reaches at least 99.4% and sensitivity averages 75.7% (min 56.4%, max 87.7%) across polarity/CE setups, matching Table ST3.
- The total pipeline (filter GUI, embedding generation, classification GUI) runs without the out-of-memory or thermal failures observed on a passively-cooled MacBook Air.
- Each labeled MS/MS spectrum (from MSnLib, the BOKU in-house database, and the wheat ear extract) yields one embedding vector of consistent dimensionality across the dataset.

## Limitations

- Embedding generation is the most time-consuming step of the AnnoMe pipeline (4 hours on a standard laptop with 12 cores, 16 GB RAM) and is the limiting factor for dataset size.
- The generation could not be completed on a less powerful, passively-cooled MacBook Air (M4, 16 GB main memory, only passive cooling).
- Embeddings must be regenerated whenever the underlying labeled MS/MS dataset changes; the cache only accelerates repeat runs on an unchanged dataset.
- Even with good embeddings, MS/MS classification into substance classes most often will not be successful, especially with highly imbalanced training datasets — the package cannot resolve this, so users must exercise caution with the results.

## Evidence

- [abstract] Structure-informed embeddings (MS2DeepScore) are computed and custom-trained classification models with a majority vote are used to classify compounds as 'relevant' or 'other': "alongside structure-informed embeddings (MS2DeepScore), the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote"
- [readme] Embedding generation is the limiting step and takes 4 hours on a standard laptop.: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop (Intel Core Ultra 5 125U, 12 cores; 16GB main memory; SSD; Windows 11)."
- [readme] Embeddings are cached on disk, reducing subsequent runs to a few minutes.: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step from hours to few minutes."
- [intro] Classifiers are trained on labeled MS/MS spectra of compounds of interest and others.: "For this, the classifiers first need to be trained on a large set of MS/MS spectra of compounds of interest (e.g., obtained from reference standards) and others"
- [discussion] Highly imbalanced training datasets are a failure mode the package cannot resolve.: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful, especially when the training dataset is highly imbalanced."
