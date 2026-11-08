---
name: cross-validated-classification-metrics-evaluation
description: Use when 'relevant'/'other'-labeled HR-MS/MS spectra (e.g., SMARTS-filtered
  MSnLib v6 MGF files split by polarity and collision energy) have MS2DeepScore embeddings
  and you must train and evaluate classifiers with balanced-accuracy metrics.
when_to_use_negative:
- The training dataset of labeled spectra is highly imbalanced — the article states
  classification will most often not succeed and the package cannot resolve this failure
  mode.
- Input spectra have not been labeled/filtered into 'relevant' and 'other' classes
  yet (the filtering GUI step is a prerequisite).
- MS2DeepScore embeddings are not available and compute resources are insufficient
  — the embedding step failed on a less powerful MacBook Air (M4, 16GB, passive cooling).
tools:
- name: AnnoMe
  role: Provides the classification GUI (annome_classificationgui) used to train LDA/NN/SVM
    classifiers with cross-validation and different random seeds and to derive majority-vote
    classifications
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings on which the classifiers are trained;
    this is the time-limiting step (~4 hours on a 12-core laptop)
- name: uv
  role: Runs the AnnoMe entry points (e.g., `uv run annome_classificationgui`) in
    the managed Python environment
  repo: https://github.com/astral-sh/uv
- name: Python
  role: Implementation language of AnnoMe in which the cross-validated classifier
    training and metric computation are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:33800dcc9be61232426d7957a4aceb8d9794877226871cdaf0056ba80c3428f7
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
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-validated-classification-metrics-evaluation

## Summary

Train multiple classifiers (LDA, NN, SVM) with cross-validation and different random seeds on MS2DeepScore embeddings of labeled HR-MS/MS spectra, aggregate them by majority vote, and report balanced accuracy, TPR, and FPR separately for training and independent test spectral subsets.

## When to use

Use when 'relevant'/'other'-labeled HR-MS/MS spectra (e.g., SMARTS-filtered MSnLib v6 MGF files split by polarity and collision energy) have MS2DeepScore embeddings and you must train and evaluate classifiers with balanced-accuracy metrics.

## When NOT to use

- The training dataset of labeled spectra is highly imbalanced — the article states classification will most often not succeed and the package cannot resolve this failure mode.
- Input spectra have not been labeled/filtered into 'relevant' and 'other' classes yet (the filtering GUI step is a prerequisite).
- MS2DeepScore embeddings are not available and compute resources are insufficient — the embedding step failed on a less powerful MacBook Air (M4, 16GB, passive cooling).

## Inputs

- SMARTS-filtered and labeled MGF MSnLib v6 spectra files ('relevant' vs 'other'), split per polarity and collision-energy subset
- MS2DeepScore structure-informed embeddings for the filtered spectra (cached on disk after first computation)
- Independent test spectra, e.g., the BOKU University in-house database
- Class-label assignments produced with the AnnoMe filtering GUI (annome_filtergui)

## Outputs

- Trained LDA, NN, and SVM classifiers with cross-validation results across random seeds
- Majority-vote 'relevant'/'other' classification for each input MS/MS spectrum
- evaluation_metrics.csv with balanced accuracy, TPR, and FPR for training and test subsets

## How to apply

Compute MS2DeepScore structure-informed embeddings for all filtered spectra (via AnnoMe's classification GUI, embeddings are cached on disk after the first run, which took ~4 hours on a 12-core laptop). Train several classifiers — LDA, neural network, and SVM — using cross-validation and different random seeds so that classifier diversity supports a robust aggregate. Aggregate the per-classifier predictions by majority vote to derive the final 'relevant'/'other' call for each input MS/MS spectrum. Evaluate on both the training subsets and an independent test set (e.g., the BOKU University in-house database), computing balanced accuracy, TPR, and FPR per polarity and collision-energy subset, and write evaluation_metrics.csv. Compare your metrics against the reported Table ST1 benchmarks: balanced accuracy 0.866 (sd 3%, min 0.818, max 0.908) on training and 0.82 (sd 4%, min 0.76, max 0.87) on the independent test set, TPR >55% training / 82% test, and FPR <10% training / 5.2% test.

## Related tools

- **AnnoMe** (Provides the classification GUI (annome_classificationgui) used to train LDA/NN/SVM classifiers with cross-validation and different random seeds and to derive majority-vote classifications) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings on which the classifiers are trained; this is the time-limiting step (~4 hours on a 12-core laptop))
- **uv** (Runs the AnnoMe entry points (e.g., `uv run annome_classificationgui`) in the managed Python environment) — https://github.com/astral-sh/uv
- **Python** (Implementation language of AnnoMe in which the cross-validated classifier training and metric computation are executed)

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- Training-data balanced accuracy across polarity and collision-energy subsets falls near 0.866 (sd 3%, min 0.818, max 0.908).
- Independent test-set balanced accuracy falls near 0.82 (sd 4%, min 0.76, max 0.87).
- TPR exceeds 55% on training subsets and reaches about 82% on the test set.
- FPR stays below 10% on training subsets and near 5.2% on the test set.
- Majority-vote of the multiple seeded classifiers is consistent, and per-seed results are stable across different random seeds within cross-validation.

## Limitations

- Classification of MS/MS spectra into substance classes is non-trivial and will most often not be successful, especially with highly imbalanced training datasets; the package cannot resolve this.
- Users are warned to exercise caution with generated results, i.e., the assignment of MS/MS spectra to specific compound classes of interest.
- The MS2DeepScore embedding step is the most time-consuming step (~4 hours on a 12-core laptop) and could not be completed on a less powerful MacBook Air (M4, 16GB RAM, passive cooling); no changelog was found for the repository.

## Evidence

- [methods] Cross-validated multi-classifier training: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [abstract] Majority-vote aggregation: "the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote"
- [methods] Classification GUI entry point: "Start the classification GUI via the command `uv run annome_classificationgui`"
- [readme] Embedding caching behavior: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [readme] Scope limitation on imbalanced data: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
