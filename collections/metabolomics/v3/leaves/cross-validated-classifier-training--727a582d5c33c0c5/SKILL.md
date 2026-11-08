---
name: cross-validated-classifier-training
description: Use when labeled 'relevant' and 'other' HR-MS/MS reference spectra (e.g.,
  MSnLib or in-house libraries) must be trained into binary classifiers whose cross-validated
  sensitivity and specificity per polarity/CE setup must be quantified.
when_to_use_negative:
- When the training dataset is highly imbalanced — the article states classification
  of MS/MS spectra into substance classes will most often not be successful in that
  case and the package cannot resolve this issue
- When only unlabeled spectra are available — this skill requires annotated and labeled
  training spectra of compounds of interest and others
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: AnnoMe
  role: Python package used to load labeled MSnLib and in-house MS/MS reference spectra
    and to train LDA/NN/SVM classifiers with cross-validation and majority vote
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates structure-informed embeddings of the training MS/MS spectra that
    serve as classifier input features; embeddings are cached on disk after first
    computation
- name: Python
  role: Implementation language for the AnnoMe training pipeline, run via uv
- name: uv
  role: Python toolkit used to install and run the AnnoMe environment (uv run)
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:2dfab74670fd9ff7b37955e50e721d5642d7331a29ceafff462e59336930778e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cross-validated-classifier-training

## Summary

Train LDA, NN, and SVM classifiers on labeled MS/MS spectra using MS2DeepScore embeddings with cross-validation across multiple random seeds, stratified separately by polarity and collision energy, then aggregate per-seed results by majority vote to report sensitivity and specificity per stratified setup.

## When to use

Use when labeled 'relevant' and 'other' HR-MS/MS reference spectra (e.g., MSnLib or in-house libraries) must be trained into binary classifiers whose cross-validated sensitivity and specificity per polarity/CE setup must be quantified.

## When NOT to use

- When the training dataset is highly imbalanced — the article states classification of MS/MS spectra into substance classes will most often not be successful in that case and the package cannot resolve this issue
- When only unlabeled spectra are available — this skill requires annotated and labeled training spectra of compounds of interest and others

## Inputs

- Labeled 'relevant' and 'other' reference HR-MS/MS spectra from MSnLib libraries and the BOKU University in-house database
- Polarity and collision energy annotations for each spectrum (used to define stratification groups)
- Random seed lists for cross-validation repetition

## Outputs

- Per-polarity/CE-setup cross-validation metrics table (sensitivity, specificity) saved as cv_metrics.csv
- Trained LDA, NN, and SVM classifier models aggregated by majority vote
- Cached MS2DeepScore embeddings on disk

## How to apply

Load the labeled 'relevant' (e.g., natural products and prenylated flavonoids) and 'other' (e.g., wheat ear extract) reference MS/MS spectra from MSnLib libraries and the in-house database using the AnnoMe package. Compute MS2DeepScore structure-informed embeddings for all training spectra and cache them on disk after the first run to avoid the hours-long recomputation. Stratify the training data separately by polarity and by collision energy, keeping each polarity/CE setup distinct rather than merging CE setups. Within each stratum, train LDA, NN, and SVM classifiers with cross-validation across different random seeds, then aggregate per-seed classifier results by majority vote and compute specificity and sensitivity for each setup. Validate against expected ranges: cross-validated specificity should reach at least 99.4% and sensitivity should average 75.7% (min 56.4%, max 87.7%), and export the per-setup metrics table.

## Related tools

- **AnnoMe** (Python package used to load labeled MSnLib and in-house MS/MS reference spectra and to train LDA/NN/SVM classifiers with cross-validation and majority vote) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates structure-informed embeddings of the training MS/MS spectra that serve as classifier input features; embeddings are cached on disk after first computation)
- **Python** (Implementation language for the AnnoMe training pipeline, run via uv)
- **uv** (Python toolkit used to install and run the AnnoMe environment (uv run)) — https://github.com/astral-sh/uv

## Examples

```
git clone https://github.com/chrboku/AnnoMe && cd AnnoMe && uv run annome_classificationgui
```

## Evaluation signals

- Cross-validated specificity for each polarity/CE setup reaches at least 99.4%, indicating nearly all 'other'-class spectra are correctly predicted
- Cross-validated sensitivity averages 75.7% across setups with per-setup values falling within the reported range of 56.4%–87.7% (Table ST3)
- Per-setup metrics table is produced as cv_metrics.csv with one row per distinct polarity/CE stratum (no merged CE setups)
- MS2DeepScore embeddings are cached on disk after the first run so subsequent runs skip the multi-hour embedding computation
- Majority-vote aggregation over per-seed classifier results yields a single 'relevant'/'other' label per input spectrum

## Limitations

- Classification of MS/MS spectra into substance classes is a non-trivial task and will most often fail, especially with highly imbalanced training datasets; the package cannot resolve this
- Users are warned to exercise caution with results assigning MS/MS spectra to specific compound classes of interest
- Generating MS2DeepScore embeddings is the limiting step, taking about 4 hours on a standard laptop; a less powerful MacBook Air could not process the same dataset
- Sensitivity is much lower than specificity (min 56.4% vs min 99.4%), so some 'relevant'-class spectra are misclassified as 'other'

## Evidence

- [methods] train classifiers with stratified cross-validation: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [methods] majority-vote aggregation: "The results are then aggregated and a majority-vote is derived which indicates the final classification"
- [readme] embedding caching: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [readme] imbalanced-data limitation: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
- [intro] training data requirement: "the classifiers first need to be trained on a large set of MS/MS spectra of compounds of interest (e.g., obtained from reference standards) and others"
