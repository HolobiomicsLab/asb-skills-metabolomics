---
name: majority-vote-ensemble-classification
description: Use when you need to prioritize novel HR-MS/MS spectra by a compound
  class of interest — e.g., flagging putative prenylated flavonoids in plant extracts
  — using curated 'relevant'/'other' reference spectra.
when_to_use_negative:
- When the training dataset is highly imbalanced between 'relevant' and 'other' compounds
  — the article and README state the package is not capable of resolving this issue
  and classification will most often not be successful
- When only a single classifier or a single random seed is available — the method
  depends on aggregating multiple cross-validated classifiers with different random
  seeds by majority vote
- When the input is raw MS1 peak data without HR-MS/MS spectra — AnnoMe operates on
  MS/MS spectra (MGF inference files)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Python package that computes MS2DeepScore embeddings, applies the trained
    classifiers to embeddings, and aggregates individual predictions by majority vote
    into the final classification
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings of MS/MS spectra that the classifiers
    consume as features
- name: uv
  role: Toolkit used to install and run AnnoMe commands (e.g., uv run annome_classificationgui)
  repo: https://github.com/astral-sh/uv#installation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:66d3f4b4346d5ad208b5dc0d457e1190807033cacca863c0a6fe342eb1b4c4cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# majority-vote-ensemble-classification

## Summary

Classify HR-MS/MS spectra into user-defined 'relevant' vs 'other' substance classes by combining predictions from multiple classifiers (LDA, NN, SVM, etc.) trained with cross-validation and different random seeds on MS2DeepScore embeddings, and aggregating the individual outcomes by majority vote.

## When to use

Use when you need to prioritize novel HR-MS/MS spectra by a compound class of interest — e.g., flagging putative prenylated flavonoids in plant extracts — using curated 'relevant'/'other' reference spectra.

## When NOT to use

- When the training dataset is highly imbalanced between 'relevant' and 'other' compounds — the article and README state the package is not capable of resolving this issue and classification will most often not be successful
- When only a single classifier or a single random seed is available — the method depends on aggregating multiple cross-validated classifiers with different random seeds by majority vote
- When the input is raw MS1 peak data without HR-MS/MS spectra — AnnoMe operates on MS/MS spectra (MGF inference files)

## Inputs

- MGF file of inference HR-MS/MS spectra (e.g., plant extract such as Glycyrrhiza uralensis roots or Paulownia tomentosa fruits)
- Curated training dataset of labeled 'relevant' and 'other' MS/MS spectra (e.g., MSnLib libraries, in-house BOKU University database)
- MS2DeepScore embeddings for the inference spectra (generated and cached by AnnoMe)
- Trained classifier models (LDA, NN, SVM, etc.) from cross-validation with different random seeds

## Outputs

- Per-spectrum 'relevant'/'other' classification labels aggregated by majority vote
- Per-sample counts and fractions of 'relevant' spectra (e.g., classification_results.csv)

## How to apply

First generate structure-informed MS2DeepScore embeddings for all inference MGF spectra in AnnoMe (these are cached on disk after the first run, which otherwise takes ~4 h on a standard laptop). Independently train several classifiers — LDA, NN, SVM, etc. — on labeled training MS/MS spectra using cross-validation and different random seeds, then apply each individual classifier to the embeddings. Aggregate the per-spectrum predictions across classifiers by majority vote to derive the final 'relevant' or 'other' label. For benchmarking, tally the number and fraction of 'relevant' spectra per sample (e.g., Glycyrrhiza uralensis: 231 (25%) and 305 (15%) spectra; Paulownia tomentosa: 171–1005 spectra, 17%–34%) and write per-spectrum labels to classification_results.csv. Rationale: no single classifier on imbalanced MS/MS training data is reliable, so an ensemble of independently seeded, cross-validated models voting on MS2DeepScore structure-informed embeddings mitigates instability.

## Related tools

- **AnnoMe** (Python package that computes MS2DeepScore embeddings, applies the trained classifiers to embeddings, and aggregates individual predictions by majority vote into the final classification) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings of MS/MS spectra that the classifiers consume as features)
- **uv** (Toolkit used to install and run AnnoMe commands (e.g., uv run annome_classificationgui)) — https://github.com/astral-sh/uv#installation

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- Reproduce reported counts: for Glycyrrhiza uralensis, 231 and 305 HR-MS/MS spectra predicted as prenylated flavonoids in the two ionization polarities; for Paulownia tomentosa, 171–1005 spectra (17%–34%)
- Verify that the majority vote aggregates results from multiple classifiers (LDA, NN, SVM, etc.) each trained with cross-validation and different random seeds
- Confirm MS2DeepScore embeddings are computed and cached on disk so subsequent classification runs complete in minutes rather than hours
- Check that the aggregated per-spectrum labels are written to classification_results.csv with per-sample tallies matching Section 3.3 / Table ST4

## Limitations

- Classification of MS/MS spectra into substance classes is a non-trivial task and will most often not be successful, especially with highly imbalanced training datasets; the package cannot resolve this issue
- Users must exercise caution with the assignment of MS/MS spectra to specific compound classes of interest
- The limiting step is generation of MS2DeepScore embeddings (~4 h on a standard laptop; not feasible on a passively cooled MacBook Air with 16 GB RAM)
- No changelog was found for the repository

## Evidence

- [methods] majority-vote aggregation: "The results are then aggregated and a majority-vote is derived which indicates the final classification"
- [methods] ensemble of classifiers: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [methods] MS2DeepScore embeddings: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [discussion] imbalance failure mode: "especially when the training dataset is highly imbalanced. The presented python package is not capable of resolving this issue."
- [results] reported prenylated flavonoid counts: [evidence span withheld — anchor status: not_found]
- [readme] package purpose: "This is a package for the classification of MS/MS spectra of novel compounds into either "relevant" or "other" compounds."
