---
name: learning-curve-performance-analysis
description: Use when planning or justifying a custom AnnoMe/MS2DeepScore classifier
  training run and you must decide whether your number of annotated 'relevant'-class
  reference spectra is sufficient or can be trimmed.
when_to_use_negative:
- When the training dataset is highly imbalanced — the AnnoMe README states classification
  of MS/MS spectra into substance classes will most often fail and the package cannot
  resolve this issue, so a learning curve will not rescue it
- When the analysis goal is filtering or prioritizing an inference extract (e.g. Paulownia
  tomentosa or Glycyrrhizza uralensis spectra) rather than assessing training-data
  sufficiency
- When embeddings have not yet been computed — MS2DeepScore generation is the time-limiting
  step (4 hours on a standard laptop), so run embedding generation first before any
  subsampling
tools:
- name: AnnoMe
  role: Trains the LDA, NN and SVM classifiers with cross-validation and different
    random seeds and aggregates their outputs into a majority vote at each training
    fraction
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings used as classifier input features;
    embeddings are cached on disk after first run
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Runs the AnnoMe commands (e.g. uv run annome_classificationgui) and manages
    the Python environment
  repo: https://github.com/astral-sh/uv#installation
- name: Python
  role: Host language for the subsampling, retraining and recall-aggregation script
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:7123dec7ee2e154036459658de62f3900e9397bba214cd81ce308a1f48a594ee
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# learning-curve-performance-analysis

## Summary

Quantifies how much annotated 'relevant'-class HR-MS/MS training data an AnnoMe classifier actually needs by randomly reducing the training set in 10% decrements (100%→50%) and tracking the 'relevant'-class recall plateau.

## When to use

Use when planning or justifying a custom AnnoMe/MS2DeepScore classifier training run and you must decide whether your number of annotated 'relevant'-class reference spectra is sufficient or can be trimmed.

## When NOT to use

- When the training dataset is highly imbalanced — the AnnoMe README states classification of MS/MS spectra into substance classes will most often fail and the package cannot resolve this issue, so a learning curve will not rescue it
- When the analysis goal is filtering or prioritizing an inference extract (e.g. Paulownia tomentosa or Glycyrrhizza uralensis spectra) rather than assessing training-data sufficiency
- When embeddings have not yet been computed — MS2DeepScore generation is the time-limiting step (4 hours on a standard laptop), so run embedding generation first before any subsampling

## Inputs

- Annotated, labeled HR-MS/MS training spectra ('relevant' and 'other' classes) from MSnLib libraries and the BOKU in-house database
- Cached MS2DeepScore structure-informed embeddings for all training and test spectra
- Held-out test HR-MS/MS spectra with known 'relevant'/'other' labels
- Random seed definitions for subsampling and cross-validation

## Outputs

- learning_curve_results.csv with one row per training fraction (100%–50%) and the corresponding 'relevant'-class recall
- Estimated minimum 'relevant'-class training-set size (~1000 spectra) at which classifier performance plateaus

## How to apply

Start from the Section 3.2 public-repository training spectra (MSnLib libraries plus the BOKU in-house database) with cached MS2DeepScore structure-informed embeddings. For each training fraction in {100%, 90%, 80%, 70%, 60%, 50%}, randomly subsample the 'relevant'-class spectra in 10% decrements with fixed random seeds, keeping the 'relevant'/'other' labels intact. At each fraction retrain the LDA, NN and SVM classifiers with cross-validation and multiple random seeds, aggregate their outputs into a majority vote, and predict 'relevant' vs 'other' for held-out test spectra. Record the 'relevant'-class recall per fraction in learning_curve_results.csv. In the AnnoMe prenylated-flavonoid study, recall stayed stable at ~62% (averaged over the first five reduction steps, 100%–50%), which underpins the derived requirement of roughly 1000 'relevant'-class spectra; accept the reproduction if recall shows no substantial degradation across all fractions.

## Related tools

- **AnnoMe** (Trains the LDA, NN and SVM classifiers with cross-validation and different random seeds and aggregates their outputs into a majority vote at each training fraction) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings used as classifier input features; embeddings are cached on disk after first run) — https://github.com/chrboku/AnnoMe
- **uv** (Runs the AnnoMe commands (e.g. uv run annome_classificationgui) and manages the Python environment) — https://github.com/astral-sh/uv#installation
- **Python** (Host language for the subsampling, retraining and recall-aggregation script)

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- learning_curve_results.csv contains one row per fraction for {100%, 90%, 80%, 70%, 60%, 50%}
- 'relevant'-class recall remains near the reported ~62% with no substantial degradation across all fractions from 100% down to 50%
- The plateau across reduction steps supports the derived requirement of roughly 1000 'relevant'-class spectra
- Subsampling at each fraction used fixed random seeds so results are reproducible across reruns
- Majority vote over LDA, NN and SVM outputs is the basis for each recall measurement, not a single classifier

## Limitations

- Classification of MS/MS spectra into substance classes is non-trivial and most often will not be successful, especially with highly imbalanced training datasets; the package cannot resolve this issue and users should exercise caution with compound-class assignments
- MS2DeepScore embedding generation is the most time-consuming step (~4 hours on a standard laptop: Intel Core Ultra 5 125U, 12 cores, 16GB RAM, SSD, Windows 11) and could not be processed on a less powerful passively cooled MacBook Air (M4, 16GB RAM)
- No changelog was found for the AnnoMe repository, so version drift in classifier behaviour should be checked against the versioned release used

## Evidence

- [supplementary] Stable recall across training-set reduction steps: [evidence span withheld — anchor status: not_found]
- [supplementary] Training-set size requirement: [evidence span withheld — anchor status: not_found]
- [readme] Classifier training and majority vote: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds. The results are then aggregated and a majority-vote is derived"
- [readme] Embedding caching behaviour: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step from hours to few minutes"
- [discussion] Imbalanced-training warning: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
