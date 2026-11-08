---
name: spectral-classifier-ensemble-training
description: Use when a curated set of annotated and labeled HR-MS/MS spectra of compounds
  of interest ('relevant') and others ('other') is available and you need a trained
  ensemble that classifies new MGF spectra into 'relevant' vs 'other' for natural-product
  prioritization.
when_to_use_negative:
- The training dataset is highly imbalanced between 'relevant' and 'other' spectra
  — the AnnoMe package explicitly cannot resolve this failure mode and classification
  will most often not be successful.
- You have no annotated, labeled training spectra of compounds of interest and others
  — the classifiers must first be trained on a large set of labeled MS/MS spectra.
- You only have few 'relevant'-class spectra (well below the ~1000-spectra regime
  the paper derived) and expect stable recall across training-set reductions.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: AnnoMe
  role: Provides the training and classification GUI that trains LDA/NN/SVM classifiers
    with cross-validation and different random seeds, aggregates majority votes, and
    caches MS2DeepScore embeddings for the labeled MGF spectra.
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Generates the structure-informed embeddings of MS/MS spectra that serve as
    the feature vectors on which the classifiers are trained; embeddings are cached
    on disk after the first run.
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Runs the AnnoMe commands (`uv run annome_filtergui`, `uv run annome_classificationgui`,
    `uv run annome_downloadresources`) that fetch resources and launch the filtering
    and classification GUIs.
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:c7340e33ba0b0c62ccf12a260e20c10c70f714e18d73eea64a0f07d5b903991f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# spectral-classifier-ensemble-training

## Summary

Train an ensemble of MS/MS spectrum classifiers (LDA, NN, SVM) with cross-validation and multiple random seeds on MS2DeepScore structure-informed embeddings, then aggregate per-classifier outputs into a majority vote that labels each HR-MS/MS spectrum as 'relevant' or 'other'. This is the core model-training step of the AnnoMe workflow for user-defined substance-class prediction.

## When to use

Use when a curated set of annotated and labeled HR-MS/MS spectra of compounds of interest ('relevant') and others ('other') is available and you need a trained ensemble that classifies new MGF spectra into 'relevant' vs 'other' for natural-product prioritization.

## When NOT to use

- The training dataset is highly imbalanced between 'relevant' and 'other' spectra — the AnnoMe package explicitly cannot resolve this failure mode and classification will most often not be successful.
- You have no annotated, labeled training spectra of compounds of interest and others — the classifiers must first be trained on a large set of labeled MS/MS spectra.
- You only have few 'relevant'-class spectra (well below the ~1000-spectra regime the paper derived) and expect stable recall across training-set reductions.

## Inputs

- Annotated and labeled HR-MS/MS spectra of 'relevant' compounds (e.g. prenylated (iso-)flavonoids from reference standards)
- Annotated and labeled HR-MS/MS spectra of 'other' compounds (reference standards of unrelated compounds and large MS/MS spectra repositories)
- MSnLib spectra libraries (MGF files) and an in-house database obtained at BOKU University
- MS2DeepScore structure-informed embeddings for all training spectra (cached on disk after first computation)

## Outputs

- Trained LDA, NN, and SVM classifiers per polarity and collision-energy spectral subset
- Majority-vote ensemble classification of each input MS/MS spectrum as 'relevant' or 'other'
- Balanced accuracy, TPR, and FPR metrics on training and independent test subsets (e.g. balanced accuracy 0.866 training / 0.82 test in the paper)

## How to apply

First prepare the labeled training corpus: annotated MS/MS spectra from reference standards and large repositories (e.g. MSnLib spectra libraries plus an in-house BOKU database), each labeled 'relevant' or 'other' per polarity and collision-energy subset. Compute MS2DeepScore structure-informed embeddings for every spectrum — this is the limiting step (≈4 hours on a 12-core laptop) and embeddings are cached on disk, so generate them once (e.g. overnight) and reuse the cache. Train several classifier types (LDA, neural network, SVM) using cross-validation with different random seeds on the embeddings, keeping the 'relevant'/'other' labels as the target. Aggregate the per-classifier predictions by majority vote to derive the final 'relevant'/'other' classification of each input MS/MS spectrum. Expect roughly 1000 'relevant'-class spectra for stable performance: reducing the 'relevant'-class training spectra in 10% steps down to 50% kept the 'relevant'-class recall near ~62%, matching the paper's training-size plateau.

## Related tools

- **AnnoMe** (Provides the training and classification GUI that trains LDA/NN/SVM classifiers with cross-validation and different random seeds, aggregates majority votes, and caches MS2DeepScore embeddings for the labeled MGF spectra.) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Generates the structure-informed embeddings of MS/MS spectra that serve as the feature vectors on which the classifiers are trained; embeddings are cached on disk after the first run.) — https://github.com/chrboku/AnnoMe
- **uv** (Runs the AnnoMe commands (`uv run annome_filtergui`, `uv run annome_classificationgui`, `uv run annome_downloadresources`) that fetch resources and launch the filtering and classification GUIs.) — https://github.com/astral-sh/uv

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- Balanced accuracy on training data should approach the reported 0.866 (sd 3%, min 0.818, max 0.908) and on the independent test set ~0.82 (sd 4%, min 0.76, max 0.87) for the (iso-)flavonoid demonstration.
- True positive rate ('relevant' correctly predicted) should be >55% on training data and ~82% on test; false positive rate should stay <10% training / 5.2% test.
- For the prenylated-flavonoid demo, the ground-truth negative control of 155 non-prenylated (iso-)flavonoid spectra should be assigned to 'other' at 58% (negative mode) and 67% (positive mode).
- Randomly reducing 'relevant'-class training spectra in 10% steps to 50% should keep 'relevant'-class recall near ~62%, confirming the ~1000-spectra training-size plateau.
- MS2DeepScore embeddings must be cached on disk after the first computation so that subsequent training runs take minutes instead of hours.

## Limitations

- Classification of MS/MS spectra into substance classes is a non-trivial task and most often will not be successful, especially with highly imbalanced training datasets — the package cannot resolve this; users must exercise caution with results.
- Generation of MS2DeepScore embeddings is the most time-consuming step (~4 hours on a standard laptop, 12 cores, 16 GB RAM, SSD); the same dataset could not be processed on a less powerful passively-cooled MacBook Air (M4, 16 GB).
- The embedding computation resource requirement scales with the number of MS/MS spectra processed; plan accordingly for large repositories.
- All public MS/MS resources downloaded by AnnoMe total approximately 11 GB (as of 2.2026), so metered or slow connections are a practical constraint.

## Evidence

- [readme] Multi-classifier cross-validated training with majority vote: "Using training data consisting of annotated and labeled MS/MS spectra of compounds of interest and others, different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and"
- [readme] Majority vote defines final classification: "The results are then aggregated and a majority-vote is derived which indicates the final classification of the input MS/MS spectra to either "relevant" or "other"."
- [readme] Embedding generation is the limiting step and is cached: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop"
- [readme] Embeddings cached on disk reduce repeat runtime from hours to minutes: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step from hours to few minutes."
- [results] Training-set-size reduction keeps 'relevant'-class recall stable (~62%): [evidence span withheld — anchor status: not_found]
- [discussion] Caution required: classification often unsuccessful with imbalanced training data: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
- [intro] AnnoMe classifies novel MS/MS spectra as 'relevant' or 'other': "This is a package for the classification of MS/MS spectra of novel compounds into either "relevant" or "other" compounds."
