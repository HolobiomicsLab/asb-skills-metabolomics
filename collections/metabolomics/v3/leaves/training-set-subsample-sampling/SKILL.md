---
name: training-set-subsample-sampling
description: Use when you need to determine the minimum number of 'relevant'-class
  MS/MS spectra required to train AnnoMe classifiers, by measuring how 'relevant'-class
  recall degrades as training data is randomly reduced.
when_to_use_negative:
- Input MS/MS spectra are already highly imbalanced between 'relevant' and 'other'
  classes — AnnoMe's README explicitly warns that classification 'will not be successful'
  in that case and the package cannot resolve it.
- The question is not classifier size sensitivity but e.g. spectrum filtering or GUI-based
  comparison — use the filtering GUI (annome_filtergui) or classification GUI instead.
- You lack a labeled 'relevant'/'other' training set of reference spectra; this skill
  requires annotated spectra of compounds of interest obtained from reference standards
  or large MS/MS spectra repositories.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AnnoMe
  role: Python package used to train the LDA/NN/SVM classifiers on subsampled 'relevant'/'other'
    spectra, aggregate their majority vote, and compute recall on the held-out test
    set at each training fraction
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings of the training and test HR-MS/MS
    spectra that serve as classifier features; embeddings are cached on disk after
    the first run
- name: uv
  role: Python package/project manager used to run AnnoMe commands (e.g. uv run annome_classificationgui)
    in the article's setup workflow
  repo: https://github.com/astral-sh/uv#installation
- name: Python
  role: Implementation language of AnnoMe and the subsampling/recall evaluation scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:d3b2dc5e950c2e470da0b7a16b93c8af850865050c199a3eaa269d6c3611cacb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# training-set-subsample-sampling

## Summary

Randomly subsample the annotated 'relevant'-class HR-MS/MS training spectra in 10% decrements (100% down to 50%) to test how sensitive AnnoMe's LDA/NN/SVM majority-vote classifier is to training-set size, and confirm the roughly 1000-spectra requirement for stable prenylated-flavonoid prediction.

## When to use

Use when you need to determine the minimum number of 'relevant'-class MS/MS spectra required to train AnnoMe classifiers, by measuring how 'relevant'-class recall degrades as training data is randomly reduced.

## When NOT to use

- Input MS/MS spectra are already highly imbalanced between 'relevant' and 'other' classes — AnnoMe's README explicitly warns that classification 'will not be successful' in that case and the package cannot resolve it.
- The question is not classifier size sensitivity but e.g. spectrum filtering or GUI-based comparison — use the filtering GUI (annome_filtergui) or classification GUI instead.
- You lack a labeled 'relevant'/'other' training set of reference spectra; this skill requires annotated spectra of compounds of interest obtained from reference standards or large MS/MS spectra repositories.

## Inputs

- Annotated and labeled HR-MS/MS training spectra (MGF) from MSnLib libraries and the BOKU in-house database
- Class labels ('relevant' vs 'other') for each training spectrum
- MS2DeepScore structure-informed embeddings for all spectra (cached on disk after first computation)
- Held-out test spectra for recall evaluation
- Random seed list used by the AnnoMe training procedure

## Outputs

- learning_curve_results.csv with one row per training fraction (100%–50%) recording 'relevant'-class recall
- Per-fraction trained classifiers (LDA, NN, SVM) with cross-validation scores
- Majority-vote classification results ('relevant' vs 'other') per fraction on the test set

## How to apply

Start from the Section 3.2 public-repository training set (MSnLib spectra libraries plus the BOKU in-house database) and compute or load the cached MS2DeepScore structure-informed embeddings for every spectrum. For each training fraction from 100% down to 50% in 10% decrements, randomly subsample the annotated training spectra while preserving the 'relevant'/'other' class labels, using fixed random seeds for reproducibility as in the AnnoMe training procedure. At each fraction retrain the classifiers (LDA, NN, SVM) with cross-validation and different random seeds, aggregate their outputs into the majority vote, and predict 'relevant' vs 'other' for the held-out test spectra. Record the 'relevant'-class recall per fraction in learning_curve_results.csv. The rationale is that the plateau — on average 62% of the 'relevant' class correctly predicted across the first five reduction steps — establishes the derived requirement of roughly 1000 'relevant'-class spectra, below which performance would deteriorate.

## Related tools

- **AnnoMe** (Python package used to train the LDA/NN/SVM classifiers on subsampled 'relevant'/'other' spectra, aggregate their majority vote, and compute recall on the held-out test set at each training fraction) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings of the training and test HR-MS/MS spectra that serve as classifier features; embeddings are cached on disk after the first run)
- **uv** (Python package/project manager used to run AnnoMe commands (e.g. uv run annome_classificationgui) in the article's setup workflow) — https://github.com/astral-sh/uv#installation
- **Python** (Implementation language of AnnoMe and the subsampling/recall evaluation scripts)

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- learning_curve_results.csv exists with one row per training fraction (100%, 90%, 80%, 70%, 60%, 50%) and a 'relevant'-class recall value for each.
- Recall at every fraction from 100% down to 50% stays near the reported plateau of ~62% (no substantial degradation), matching Section 3.4 / SI8.
- The number of 'relevant'-class spectra at the 50% fraction corresponds to roughly the derived ~1000-spectra requirement.
- Subsampling uses fixed random seeds and preserves the 'relevant'/'other' label proportions, so repeated runs reproduce the same fractions.
- Embeddings are loaded from disk cache rather than recomputed, keeping the run within minutes after the initial MS2DeepScore embedding generation.

## Limitations

- Classification of MS/MS spectra into substance classes is non-trivial and often unsuccessful when the training dataset is highly imbalanced; AnnoMe cannot resolve this issue and users must exercise caution.
- The MS2DeepScore embedding generation is the most time-consuming step (~4 hours on a standard laptop with Intel Core Ultra 5 125U, 12 cores, 16GB RAM) and cannot be completed on weaker hardware (e.g. a passive-cooled MacBook Air).
- The ~1000-spectra requirement and 62% recall plateau were established on the prenylated-flavonoid demonstration task and may not transfer to other substance classes.

## Evidence

- [results] Classifier performance remained stable across the reduction steps: on average 62% of the 'relevant' class was correctly predicted for the first five reduction steps, i.e. from 100% down to 50% of the HR-MS/MS spectra used as 'relevant' for training.: [evidence span withheld — anchor status: not_found]
- [readme] different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds. The results are then aggregated and a majority-vote is derived which indicates the final classification: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [readme] Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful, especially when the training dataset is highly imbalanced.: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
- [readme] Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step from hours to few minutes.: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [readme] For this, the classifiers first need to be trained on a large set of MS/MS spectra of compounds of interest (e.g., obtained from reference standards) and others: "the classifiers first need to be trained on a large set of MS/MS spectra of compounds of interest (e.g., obtained from reference standards)"
