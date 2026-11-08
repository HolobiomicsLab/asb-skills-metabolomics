---
name: prenylated-flavonoid-spectrum-annotation
description: Use when you need to count or prioritize putative prenylated flavonoid
  spectra in HR-MS/MS inference datasets (e.g., MGF files from Glycyrrhiza uralensis
  root or Paulownia tomentosa fruit extracts) using custom-trained spectrum classifiers.
when_to_use_negative:
- Your training dataset of 'relevant' vs 'other' spectra is highly imbalanced — the
  article explicitly warns classification will most often fail and AnnoMe cannot resolve
  this issue.
- Your target substance class is not covered by a trained classifier set — the classifiers
  must first be trained on a large set of MS/MS spectra of compounds of interest and
  others.
- Your input is already a feature table or metabolite list rather than raw/processed
  HR-MS/MS spectra in MGF format — AnnoMe operates on MS/MS spectra.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AnnoMe
  role: Python package hosting the classification GUI that applies trained LDA/NN/SVM
    classifiers to MS2DeepScore embeddings and derives majority-vote 'relevant'/'other'
    labels for each MS/MS spectrum
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates structure-informed embeddings of the inference spectra, which are
    the feature vectors the classifiers operate on; embeddings are cached on disk
    after first computation
- name: uv
  role: Python package manager used to launch the AnnoMe filtering and classification
    GUIs (e.g., `uv run annome_classificationgui`)
  repo: https://github.com/astral-sh/uv#installation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:4fbc1ad16be9278768cd4a7cd190f93d10c007a2b8aed730ffef54293e451d98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# prenylated-flavonoid-spectrum-annotation

## Summary

Classify HR-MS/MS spectra from plant extracts as 'relevant' (prenylated flavonoids) vs 'other' using AnnoMe's pre-trained classifiers (LDA, NN, SVM) applied to MS2DeepScore structure-informed embeddings with majority-vote aggregation.

## When to use

Use when you need to count or prioritize putative prenylated flavonoid spectra in HR-MS/MS inference datasets (e.g., MGF files from Glycyrrhiza uralensis root or Paulownia tomentosa fruit extracts) using custom-trained spectrum classifiers.

## When NOT to use

- Your training dataset of 'relevant' vs 'other' spectra is highly imbalanced — the article explicitly warns classification will most often fail and AnnoMe cannot resolve this issue.
- Your target substance class is not covered by a trained classifier set — the classifiers must first be trained on a large set of MS/MS spectra of compounds of interest and others.
- Your input is already a feature table or metabolite list rather than raw/processed HR-MS/MS spectra in MGF format — AnnoMe operates on MS/MS spectra.

## Inputs

- MGF files of inference HR-MS/MS spectra (deposited Glycyrrhiza uralensis root and Paulownia tomentosa fruit extract spectra)
- Pre-trained prenylated-flavonoid classifiers (LDA, NN, SVM, etc.)
- AnnoMe package installed via git clone or versioned release

## Outputs

- Per-spectrum 'relevant'/'other' classification labels
- Per-sample counts and fractions of 'relevant' spectra (e.g., classification_results.csv)
- Comparison table against reported benchmark values (Section 3.3 / Table ST4)

## How to apply

Load the deposited MGF inference spectra (Glycyrrhiza uralensis root and/or Paulownia tomentosa fruit extracts) together with the pre-trained prenylated-flavonoid classifiers (LDA, NN, SVM, etc.) into AnnoMe's classification GUI. AnnoMe computes MS2DeepScore structure-informed embeddings for all inference spectra — this is the most time-consuming step (~4 hours on a standard laptop) but embeddings are cached on disk after the first run, reducing it to minutes. Apply each individual trained classifier to the embeddings and aggregate per-spectrum predictions by majority vote to derive the final 'relevant'/'other' label for every MS/MS spectrum. Tally the number and fraction of 'relevant' spectra per sample and per ionization polarity, then compare against the reported benchmarks (Glycyrrhiza uralensis: 231 (25%) and 305 (15%) spectra; Paulownia tomentosa: 171–1005 spectra, 17%–34%). Exercise caution with the class assignments, as classification of MS/MS spectra into substance classes is non-trivial.

## Related tools

- **AnnoMe** (Python package hosting the classification GUI that applies trained LDA/NN/SVM classifiers to MS2DeepScore embeddings and derives majority-vote 'relevant'/'other' labels for each MS/MS spectrum) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates structure-informed embeddings of the inference spectra, which are the feature vectors the classifiers operate on; embeddings are cached on disk after first computation)
- **uv** (Python package manager used to launch the AnnoMe filtering and classification GUIs (e.g., `uv run annome_classificationgui`)) — https://github.com/astral-sh/uv#installation

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- Glycyrrhiza uralensis counts reproduce the reported values: 231 (25%) spectra in one ionization polarity and 305 (15%) in the other classified as prenylated flavonoids.
- Paulownia tomentosa counts fall within the reported range of 171–1005 'relevant' spectra (17%–34% across samples).
- Embedding cache exists on disk after the first run, so the second run's MS2DeepScore step completes in minutes rather than hours.
- Every inference spectrum in the MGF file receives exactly one final 'relevant'/'other' label from the majority vote across all classifiers.
- Per-sample fractions of 'relevant' spectra are computed for both ionization polarities separately and recorded in the output CSV.

## Limitations

- Classification of MS/MS spectra into substance classes is a non-trivial task and most often will not be successful, especially with highly imbalanced training datasets; AnnoMe cannot resolve this failure mode.
- Users should exercise caution with the assignment of MS/MS spectra to specific compound classes of interest.
- MS2DeepScore embedding generation is resource-intensive: ~4 hours on a standard laptop (Intel Core Ultra 5 125U, 12 cores, 16GB RAM); it failed on a less powerful MacBook Air (M4, 16GB RAM, passive cooling).

## Evidence

- [methods] majority-vote aggregation of classifier results: "The results are then aggregated and a majority-vote is derived which indicates the final classification"
- [methods] MS2DeepScore embeddings cached on disk: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [readme] inference datasets are prenylated-flavonoid-producing plants: "The inference dataset are obtained from extracts of plants that are known producers of prenylated flavonoid compounds."
- [readme] classification GUI invocation: "Start the classification GUI via the command `uv run annome_classificationgui`"
- [discussion] caution warning on imbalanced training data: "especially when the training dataset is highly imbalanced. The presented python package is not capable of resolving this issue."
