---
name: msms-spectra-annotation
description: Use when experimental HR-MS/MS spectra (MGF) must be classified into
  user-defined substance classes, e.g. prioritizing putative (prenylated) flavonoids
  in natural product discovery.
when_to_use_negative:
- The training dataset is highly imbalanced — the paper explicitly states classification
  will most often fail in that case and the package cannot resolve it
- The goal is de novo structure elucidation rather than binary 'relevant'/'other'
  prioritization of spectra against user-defined substance classes
- Input spectra are already labeled with trusted class annotations, making substructure
  filtering and classifier training unnecessary
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Python package providing the filtering GUI (annome_filtergui) for SMARTS-based
    MGF substructure filtering and the classification GUI (annome_classificationgui)
    for MS2DeepScore embedding, classifier training, and majority-vote prediction
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Computes structure-informed spectral embeddings that are the input features
    for the trained classifiers; the most time-consuming, disk-cached step
- name: uv
  role: Python package/runner used to launch AnnoMe entry points (uv run annome_downloadresources,
    annome_filtergui, annome_classificationgui)
  repo: https://github.com/astral-sh/uv
- name: git
  role: Used to obtain a copy of the AnnoMe repository (git clone https://github.com/chrboku/AnnoMe)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:14cd8a0d5793507bb3951ae914683c8de1ac773f72a7b072a78cc3c8b2fc6521
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - git
  - uv
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# msms-spectra-annotation

## Summary

Train user-defined 'relevant'/'other' classifiers for HR-MS/MS spectra using AnnoMe: substructure (SMARTS) filtering of MGF spectral libraries to build training classes, MS2DeepScore embeddings, and a majority vote over LDA/NN/SVM classifiers to assign novel spectra to substance classes.

## When to use

Use when experimental HR-MS/MS spectra (MGF) must be classified into user-defined substance classes, e.g. prioritizing putative (prenylated) flavonoids in natural product discovery.

## When NOT to use

- The training dataset is highly imbalanced — the paper explicitly states classification will most often fail in that case and the package cannot resolve it
- The goal is de novo structure elucidation rather than binary 'relevant'/'other' prioritization of spectra against user-defined substance classes
- Input spectra are already labeled with trusted class annotations, making substructure filtering and classifier training unnecessary

## Inputs

- MGF-format MS/MS spectral library (e.g. MSnLib v6) as training input for the filtering GUI
- SMARTS substructure query strings defining the 'relevant' substance class (e.g. (iso-)flavonoid, prenylated flavonoid, prenylated chalcone SMARTS)
- Experimental HR-MS/MS spectra in MGF format for classification (inference datasets such as Paulownia tomentosa fruit or Glycyrrhizza uralensis root extracts)

## Outputs

- Filtered MGF files split into 'relevant' and 'other' classes (e.g. 655 (iso-)flavonoid vs 10000 other spectra)
- MS2DeepScore embeddings for all spectra, cached on disk
- Majority-vote classification labels ('relevant'/'other') for each input MS/MS spectrum
- Summary table of class counts from the filtering step

## How to apply

Install uv and clone the AnnoMe repository, then download public MS/MS resources (~11 GB, resumable) via `uv run annome_downloadresources`. Build the training classes by launching the filtering GUI (`uv run annome_filtergui`), loading the MSnLib v6 spectral library as the input MGF dataset, and applying (iso-)flavonoid SMARTS substructure queries so matching structures are labeled 'relevant' and the remainder 'other' (reported split: 655 relevant vs 10000 other spectra); repeat with prenylated-flavonoid and prenylated-chalcone SMARTS definitions for the Section 3.2 split. Export the filtered MGF files, then in the classification GUI (`uv run annome_classificationgui`) compute MS2DeepScore embeddings (cached on disk after the first run; ~4 h on a standard laptop), train several classifiers (LDA, NN, SVM) with cross-validation and different random seeds, and aggregate their outputs into a majority vote. Interpret the majority-vote label as the final 'relevant'/'other' assignment and validate against the reported class counts before trusting predictions.

## Related tools

- **AnnoMe** (Python package providing the filtering GUI (annome_filtergui) for SMARTS-based MGF substructure filtering and the classification GUI (annome_classificationgui) for MS2DeepScore embedding, classifier training, and majority-vote prediction) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Computes structure-informed spectral embeddings that are the input features for the trained classifiers; the most time-consuming, disk-cached step)
- **uv** (Python package/runner used to launch AnnoMe entry points (uv run annome_downloadresources, annome_filtergui, annome_classificationgui)) — https://github.com/astral-sh/uv
- **git** (Used to obtain a copy of the AnnoMe repository (git clone https://github.com/chrboku/AnnoMe))

## Examples

```
uv run annome_filtergui
```

## Evaluation signals

- The (iso-)flavonoid SMARTS filter on MSnLib v6 yields exactly 655 'relevant' and 10000 'other' spectra, matching Section 3.1
- Repeated filtering with prenylated-flavonoid and prenylated-chalcone SMARTS definitions reproduces the Section 3.2 'relevant'/'other' split
- MS2DeepScore embeddings are present on disk after the first run and re-running the step completes in minutes rather than hours
- Classifier cross-validation across different random seeds and a stable majority vote agree on 'relevant'/'other' labels for inference spectra
- Filtered output is valid MGF and every spectrum carries exactly one class label ('relevant' or 'other')

## Limitations

- Classification of MS/MS spectra into substance classes will most often not succeed, especially with highly imbalanced training datasets; the package cannot resolve this
- Users must exercise caution with assignments of MS/MS spectra to specific compound classes
- The resource download is ~11 GB and the MS2DeepScore embedding step took 4 hours on a standard laptop; a less powerful MacBook Air (M4, 16 GB, passive cooling) could not process the same dataset
- No changelog was found for the repository

## Evidence

- [readme] This is a package for the classification of MS/MS spectra of novel compounds into either "relevant" or "other" compounds.: "This is a package for the classification of MS/MS spectra of novel compounds into either "relevant" or "other" compounds."
- [methods] Start the filtering GUI via the command `uv run annome_filtergui`: "Start the filtering GUI via the command `uv run annome_filtergui`"
- [methods] different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [readme] The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop"
- [discussion] Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
