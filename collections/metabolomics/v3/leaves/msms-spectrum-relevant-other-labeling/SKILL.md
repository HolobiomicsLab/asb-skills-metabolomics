---
name: msms-spectrum-relevant-other-labeling
description: Use when HR-MS/MS spectra of novel compounds must be labeled 'relevant'
  vs 'other' for a user-defined substance class (e.g., prenylated (iso-)flavonoids)
  from annotated training spectra and MS2DeepScore embeddings.
when_to_use_negative:
- The training set of 'relevant' reference spectra is highly imbalanced versus 'other'
  spectra — the authors state classification will most often not succeed in this case
  and AnnoMe cannot resolve it.
- Compute is too limited to generate and cache MS2DeepScore embeddings (the ~4-hour
  step could not be processed on a 16GB MacBook Air with passive cooling) and no cached
  embeddings exist.
- The task needs multi-class substance-class assignment rather than the binary 'relevant'/'other'
  decision AnnoMe's classifiers produce.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: AnnoMe
  role: Python package providing resource download, MGF filtering GUI, and classification
    GUI; trains the LDA/NN/SVM ensemble and disk-caches the MS2DeepScore embeddings
    used to assign 'relevant'/'other' labels
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Computes the structure-informed embeddings of the HR-MS/MS spectra that serve
    as input features for the classifiers
- name: uv
  role: Runs the AnnoMe entry points (uv run annome_downloadresources, annome_filtergui,
    annome_classificationgui) that execute the labeling workflow
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:b48022490efe643e38bbaa42698c42e9004140c2dc80704b8b5a4ad93233f7f7
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# msms-spectrum-relevant-other-labeling

## Summary

Labels HR-MS/MS spectra of novel compounds as 'relevant' or 'other' by training LDA, NN, and SVM classifiers on MS2DeepScore structure-informed embeddings of user-curated reference spectra and taking a majority vote, enabling prioritization of e.g. prenylated (iso-)flavonoids in natural product discovery.

## When to use

Use when HR-MS/MS spectra of novel compounds must be labeled 'relevant' vs 'other' for a user-defined substance class (e.g., prenylated (iso-)flavonoids) from annotated training spectra and MS2DeepScore embeddings.

## When NOT to use

- The training set of 'relevant' reference spectra is highly imbalanced versus 'other' spectra — the authors state classification will most often not succeed in this case and AnnoMe cannot resolve it.
- Compute is too limited to generate and cache MS2DeepScore embeddings (the ~4-hour step could not be processed on a 16GB MacBook Air with passive cooling) and no cached embeddings exist.
- The task needs multi-class substance-class assignment rather than the binary 'relevant'/'other' decision AnnoMe's classifiers produce.

## Inputs

- Filtered, annotated MGF training spectra from public MSnLib repositories (30,703 spectra labeled 'relevant' prenylated (iso-)flavonoids vs 'other')
- MS2DeepScore structure-informed embeddings of the training spectra, cached on disk by AnnoMe
- Independent HR-MS/MS test set: 64 prenylated-flavonoid 'relevant' spectra plus wheat ear extract 'other' spectra (7,312 total) in negative and positive ion modes
- Ground-truth negative-control spectra of 155 non-prenylated (iso-)flavonoids

## Outputs

- Majority-vote 'relevant'/'other' label for every input HR-MS/MS spectrum (per ion mode)
- Per-mode test-set accuracy metrics corresponding to Table 1 / Table ST2
- classification_results_table.tsv holding test-set accuracies and negative-control correct-'other' rates

## How to apply

Load the filtered annotated MSnLib training spectra (30,703 spectra of prenylated (iso-)flavonoids labeled 'relevant' and other compounds labeled 'other') together with their MS2DeepScore embeddings, which AnnoMe caches on disk after the first ~4-hour computation so subsequent runs take minutes. Train several classifier types — LDA, neural network, and SVM — on these embeddings using cross-validation with different random seeds so individual model errors decorrelate. Aggregate the per-classifier predictions by majority vote to derive the final 'relevant'/'other' label for each HR-MS/MS spectrum. Evaluate the ensemble per ion mode on the independent test set (64 prenylated-flavonoid 'relevant' spectra plus wheat ear extract 'other' spectra, 7,312 spectra total) and run the ground-truth negative control by classifying 155 non-prenylated (iso-)flavonoid spectra to quantify correct-'other' assignments. Write test-set accuracies and negative-control rates per ion mode to classification_results_table.tsv, validating against the expected 58% (negative mode) and 67% (positive mode) correct-'other' rates.

## Related tools

- **AnnoMe** (Python package providing resource download, MGF filtering GUI, and classification GUI; trains the LDA/NN/SVM ensemble and disk-caches the MS2DeepScore embeddings used to assign 'relevant'/'other' labels) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Computes the structure-informed embeddings of the HR-MS/MS spectra that serve as input features for the classifiers)
- **uv** (Runs the AnnoMe entry points (uv run annome_downloadresources, annome_filtergui, annome_classificationgui) that execute the labeling workflow) — https://github.com/astral-sh/uv

## Examples

```
git clone https://github.com/chrboku/AnnoMe && cd AnnoMe && uv run annome_classificationgui
```

## Evaluation signals

- Negative-control correct-'other' rates on the 155 non-prenylated (iso-)flavonoid spectra reproduce 58% (negative ion mode) and 67% (positive ion mode).
- Independent test-set metrics on the 7,312-spectrum test set match the per-mode values of Table 1 / Table ST2.
- Final label per spectrum equals the majority vote across LDA, NN, and SVM classifiers trained with cross-validation under different random seeds, and is strictly 'relevant' or 'other'.
- MS2DeepScore embeddings for training and test spectra are cached on disk, so a re-classification run completes in minutes rather than the hours needed for first-time embedding generation.
- classification_results_table.tsv contains test-set accuracies and negative-control rates for both negative and positive ion modes with no missing mode rows.

## Limitations

- Classification of MS/MS spectra into substance classes is non-trivial and most often unsuccessful, especially with highly imbalanced training datasets; the package is not capable of resolving this failure mode.
- Users must exercise caution with the generated assignments of MS/MS spectra to specific compound classes of interest.
- MS2DeepScore embedding generation is the limiting step (~4 hours on a standard 12-core laptop with 16GB RAM) and the same dataset could not be processed on a MacBook Air (M4, 16GB, passive cooling).

## Evidence

- [abstract] majority-vote ensemble over custom classifiers: "alongside structure-informed embeddings (MS2DeepScore), the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote"
- [readme] binary relevant/other task definition: "This is a package for the classification of MS/MS spectra of novel compounds into either "relevant" or "other" compounds."
- [methods] classifier training protocol: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [readme] majority-vote aggregation step: "The results are then aggregated and a majority-vote is derived which indicates the final classification"
- [methods] embedding caching behavior: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [readme] failure mode under imbalanced training data: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
