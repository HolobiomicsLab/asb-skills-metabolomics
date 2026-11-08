---
name: negative-control-classification-evaluation
description: 'Use when a trained MS/MS classifier''s false-positive rate must be quantified:
  i.e., when a curated ground-truth set of known non-target (negative-control) compound
  spectra is available and the fraction correctly assigned to the ''other'' class
  needs to be reported per ion mode.'
when_to_use_negative:
- The training dataset is highly imbalanced — the article states classification into
  substance classes will often not be successful and the package cannot resolve this
  failure mode
- The goal is compound identification or dereplication of individual spectra rather
  than binary 'relevant'/'other' class-assignment evaluation of a user-defined substance
  class
- No curated ground-truth negative-control spectra (verified non-prenylated compounds)
  exist, so the correct-'other' rejection rate cannot be measured
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Python package whose classification GUI runs the trained LDA/NN/SVM majority-vote
    classifiers on the negative-control spectra and provides MS2DeepScore embedding
    computation with disk caching
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings of the HR-MS/MS spectra on which
    the classifiers operate; embeddings are cached on disk after first computation
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Toolkit used to install and execute the AnnoMe package and its GUI commands
  repo: https://github.com/astral-sh/uv
- name: Python
  role: Language of the AnnoMe package; used to aggregate classifier predictions by
    majority vote and write classification_results_table.tsv
  repo: https://github.com/chrboku/AnnoMe
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:56eb73f7c9aaa3de2c07f41f21c88c714785f32fbf931014077eea0c2da8d0a4
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# negative-control-classification-evaluation

## Summary

Evaluates custom-trained HR-MS/MS classifiers by classifying a ground-truth set of known non-target compounds and measuring how often they are correctly rejected to the 'other' class. For the AnnoMe prenylated flavonoid model, this quantifies false-positive propensity of the LDA/NN/SVM majority-vote ensemble per ion mode.

## When to use

Use when a trained MS/MS classifier's false-positive rate must be quantified: i.e., when a curated ground-truth set of known non-target (negative-control) compound spectra is available and the fraction correctly assigned to the 'other' class needs to be reported per ion mode.

## When NOT to use

- The training dataset is highly imbalanced — the article states classification into substance classes will often not be successful and the package cannot resolve this failure mode
- The goal is compound identification or dereplication of individual spectra rather than binary 'relevant'/'other' class-assignment evaluation of a user-defined substance class
- No curated ground-truth negative-control spectra (verified non-prenylated compounds) exist, so the correct-'other' rejection rate cannot be measured

## Inputs

- Ground-truth negative-control set: 155 HR-MS/MS spectra of non-prenylated (iso-)flavonoids, annotated and labeled as 'other'
- Cached MS2DeepScore embeddings for the negative-control spectra (computed via the AnnoMe package)
- Trained classifier ensemble (LDA, NN, SVM with majority vote) built from 30703 filtered annotated MSnLib training spectra
- Independent test set for context: 64 prenylated flavonoid 'relevant' spectra plus 7312 wheat ear extract 'other' spectra
- Ion-mode annotation (positive/negative) for every spectrum in the negative-control set

## Outputs

- classification_results_table.tsv with per-mode test-set accuracies and negative-control correct-'other' assignment rates
- Negative-control correct-'other' rates per ion mode (expected: 58% negative mode, 67% positive mode)
- Per-classifier and majority-vote ensemble assignments for each of the 155 negative-control spectra

## How to apply

Assemble an independent negative-control set of HR-MS/MS spectra from compounds with verified absence of the structural feature of interest — here 155 spectra of non-prenylated (iso-)flavonoids — and run them through the same classification pipeline used for the test set: MS2DeepScore embeddings, then the ensemble of LDA, NN, and SVM classifiers trained with cross-validation and different random seeds on the 30703 filtered annotated MSnLib spectra, aggregated by majority vote. Split the results by ion mode (positive vs. negative), since classifier behavior differs per mode, and compute the fraction of spectra assigned to 'other' rather than 'relevant'. For the prenylated flavonoid models the expected correct-'other' rates are 58% in negative ion mode and 67% in positive ion mode (Section 3.2 / Table ST2). Also report the corresponding test-set accuracies (64 'relevant' prenylated flavonoid spectra plus 7312 wheat ear extract 'other' spectra, per Table 1) alongside the negative-control rates. Write all per-mode metrics (test-set accuracies and negative-control correct-'other' rates) to classification_results_table.tsv. Interpret rates well below 100% as an expected consequence of the difficulty of MS/MS-to-substance-class assignment, and exercise caution with any downstream 'relevant' assignments.

## Related tools

- **AnnoMe** (Python package whose classification GUI runs the trained LDA/NN/SVM majority-vote classifiers on the negative-control spectra and provides MS2DeepScore embedding computation with disk caching) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings of the HR-MS/MS spectra on which the classifiers operate; embeddings are cached on disk after first computation) — https://github.com/chrboku/AnnoMe
- **uv** (Toolkit used to install and execute the AnnoMe package and its GUI commands) — https://github.com/astral-sh/uv
- **Python** (Language of the AnnoMe package; used to aggregate classifier predictions by majority vote and write classification_results_table.tsv) — https://github.com/chrboku/AnnoMe

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- The correct-'other' assignment rate for the 155 non-prenylated (iso-)flavonoid spectra must reproduce 58% in negative ion mode and 67% in positive ion mode (Section 3.2 / Table ST2)
- Test-set metrics must match Table 1 / Table ST2 (64 'relevant' spectra plus 7312 wheat ear extract 'other' spectra, per-mode accuracies)
- Predictions must come from the majority-vote ensemble (LDA, NN, SVM trained with cross-validation and different random seeds), not a single classifier
- classification_results_table.tsv must be written and contain per-ion-mode breakdowns for both test-set and negative-control rates
- Negative-control spectra must be classified independently of the training set (no leakage from the 30703 filtered MSnLib training spectra)

## Limitations

- Classification of MS/MS spectra into substance classes is a non-trivial task and most often will not be successful, especially with highly imbalanced training datasets; the package cannot resolve this
- Correct-'other' rates are far from 100% (58%/67%), so a substantial fraction of known negatives is misassigned as 'relevant' — users must exercise caution with the generated class assignments
- MS2DeepScore embedding generation is the most time-consuming step (~4 hours on a standard laptop) and could not be completed on a less powerful MacBook Air (M4, 16GB RAM, passive cooling)
- The public MS/MS resources downloaded by AnnoMe are approximately 11GB, which matters for metered connections

## Evidence

- [abstract] majority-vote ensemble prediction is the reported output: "the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote"
- [methods] classifier training procedure: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [methods] embedding caching: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [readme] classification difficulty caveat: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
- [readme] ground-truth datasets used for training and testing/validation: "The ground truth datasets used for training and testing/validation are"
