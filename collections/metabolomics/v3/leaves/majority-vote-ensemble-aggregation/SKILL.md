---
name: majority-vote-ensemble-aggregation
description: Use when several independently trained classifiers (e.g., LDA, NN, SVM
  across different random seeds) each produce per-spectrum MS/MS predictions that
  must be merged into one binary 'relevant'/'other' label.
when_to_use_negative:
- When only a single classifier (or a single random seed) is available — majority
  voting requires multiple independent predictions to aggregate
- When the training dataset is highly imbalanced — the article explicitly warns that
  classification will most often fail and 'The presented python package is not capable
  of resolving this issue'
- When the task requires multi-class substance-class assignment — AnnoMe's ensemble
  produces only a binary 'relevant' vs 'other' classification
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: AnnoMe
  role: Provides the classification GUI that trains LDA/NN/SVM classifiers with cross-validation
    and different random seeds and derives the majority-vote final classification
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings on which the ensemble members'
    per-spectrum predictions are computed
- name: uv
  role: Runtime used to launch the AnnoMe classification GUI (uv run annome_classificationgui)
    that performs the training and aggregation
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:47f17cbec0fca844825bef993c0fc848424216644c2fc69e56b96ba8e9fd3a76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# majority-vote-ensemble-aggregation

## Summary

Aggregates per-classifier predictions from multiple models (LDA, NN, SVM) trained with cross-validation and different random seeds into a single final 'relevant'/'other' label per HR-MS/MS spectrum, as implemented in AnnoMe for natural product discovery.

## When to use

Use when several independently trained classifiers (e.g., LDA, NN, SVM across different random seeds) each produce per-spectrum MS/MS predictions that must be merged into one binary 'relevant'/'other' label.

## When NOT to use

- When only a single classifier (or a single random seed) is available — majority voting requires multiple independent predictions to aggregate
- When the training dataset is highly imbalanced — the article explicitly warns that classification will most often fail and 'The presented python package is not capable of resolving this issue'
- When the task requires multi-class substance-class assignment — AnnoMe's ensemble produces only a binary 'relevant' vs 'other' classification

## Inputs

- Per-spectrum class predictions ('relevant'/'other') from LDA classifiers trained with cross-validation and different random seeds
- Per-spectrum predictions from neural-network classifiers on MS2DeepScore embeddings
- Per-spectrum predictions from SVM classifiers on MS2DeepScore embeddings
- Labeled 'relevant'/'other' training spectra (e.g., MSnLib spectra filtered by SMARTS queries plus BOKU in-house database spectra)
- Independent test spectra (e.g., 64 prenylated flavonoid 'relevant' spectra plus wheat ear extract 'other' spectra)

## Outputs

- Final 'relevant' or 'other' classification for each input MS/MS spectrum
- evaluation_metrics.csv with balanced accuracy, TPR, and FPR on training and test spectral subsets
- classification_results_table.tsv with per-mode test-set accuracies and negative-control rates
- cv_metrics.csv with per-stratum specificity and sensitivity

## How to apply

First train different classifiers (LDA, NN, SVM) on MS2DeepScore structure-informed embeddings of labeled 'relevant'/'other' HR-MS/MS spectra, using cross-validation and different random seeds, stratified separately by polarity and collision energy. Each classifier then predicts 'relevant' or 'other' for every input MS/MS spectrum. Aggregate the per-classifier, per-seed predictions by majority vote: the class chosen by more than half of the ensemble members becomes the final classification for that spectrum. The rationale is that majority voting over diverse classifiers reduces seed- and model-specific variance in the embedding-based decision. Validate the aggregated output by computing balanced accuracy, TPR, and FPR per polarity/collision-energy subset and comparing against reported benchmarks (e.g., balanced accuracy 0.866 training / 0.82 test; TPR 82% and FPR 5.2% on test data). Write the per-setup metrics to evaluation_metrics.csv, classification_results_table.tsv, or cv_metrics.csv as appropriate.

## Related tools

- **AnnoMe** (Provides the classification GUI that trains LDA/NN/SVM classifiers with cross-validation and different random seeds and derives the majority-vote final classification) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings on which the ensemble members' per-spectrum predictions are computed)
- **uv** (Runtime used to launch the AnnoMe classification GUI (uv run annome_classificationgui) that performs the training and aggregation) — https://github.com/astral-sh/uv

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- The ensemble output must be binary ('relevant'/'other') for every input MS/MS spectrum, with no spectrum left unclassified
- Balanced accuracy on the independent test set should fall near the reported 0.82 (sd 4%, min 0.76, max 0.87) across polarity and collision-energy subsets
- Negative-control check: 155 non-prenylated (iso-)flavonoid spectra should be assigned to 'other' at ~58% (negative mode) and ~67% (positive mode)
- Cross-validated specificity should reach at least 99.4% and sensitivity average 75.7% (min 56.4%, max 87.7%) for the in-house-library-trained setup
- Per-fraction recall in the training-set-size reduction analysis should plateau near ~62% for 'relevant' class from 100% down to 50% of training spectra

## Limitations

- Classification of MS/MS spectra into substance classes is non-trivial and often unsuccessful, especially with highly imbalanced training datasets; the package cannot resolve this
- Users are warned to exercise caution with the assignment of MS/MS spectra to specific compound classes of interest
- Ensemble members must be trained within the same polarity/collision-energy stratum; merging CE setups is not supported by the reported workflow
- The negative-control result (only 58–67% of non-prenylated flavonoids correctly called 'other') shows the ensemble can misassign challenging near-class spectra

## Evidence

- [methods] Training then majority vote: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [methods] Aggregation to final label: "The results are then aggregated and a majority-vote is derived which indicates the final classification"
- [abstract] Abstract-level description of the ensemble: "the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote"
- [readme] README methodology: "a majority-vote is derived which indicates the final classification of the input MS/MS spectra to either "relevant" or "other""
- [readme] Imbalance limitation: "especially when the training dataset is highly imbalanced. The presented python package is not capable of resolving this issue."
