---
name: msms-spectra-compound-class-prediction
description: Use when experimental HR-MS/MS spectra (MGF files) must be screened against
  a user-defined 'relevant' substance class (e.g. prenylated flavonoids) for prioritization,
  and you have curated 'relevant' and 'other' reference spectra to train classifiers.
when_to_use_negative:
- The training dataset is highly imbalanced — the article states classification of
  MS/MS spectra into substance classes 'most often will not be successful' in this
  case and the package 'is not capable of resolving this issue'
- The goal is general-purpose compound-class ontology assignment (e.g. full ClassyFire
  hierarchy) or molecular formula/structure elucidation rather than screening against
  one user-defined 'relevant' class
- The input is already a labeled feature table or classified result rather than raw
  MS/MS spectra needing class assignment
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: AnnoMe
  role: Python package providing the filtering and classification GUIs used to filter
    MGF files, train the LDA/NN/SVM classifiers, and derive majority-vote 'relevant'/'other'
    labels
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Computes the structure-informed embeddings of MS/MS spectra that serve as
    classifier input features; embeddings are cached on disk
- name: SIRIUS
  role: 'Baseline comparator: version 6.2.2 with ClassyFire ''most specific class''
    prediction run on the same test spectra to benchmark AnnoMe''s classification
    fractions'
- name: uv
  role: Python package/project manager used to install AnnoMe and run its resource
    download, filtering GUI, and classification GUI commands
  repo: https://github.com/astral-sh/uv#installation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:1b179edf80b6e9159001d25aa82e9be8b50b16122c6072c9399b6f074719aa81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - SIRIUS
  - uv
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# msms-spectra-compound-class-prediction

## Summary

Classifies experimental HR-MS/MS spectra into a user-defined substance class of interest ('relevant', e.g. prenylated flavonoids) versus 'other' using MS2DeepScore structure-informed embeddings and custom-trained classifiers (LDA, NN, SVM) aggregated by majority vote. Serves as a user-targeted alternative/complement to generic compound-class predictors such as SIRIUS with ClassyFire.

## When to use

Use when experimental HR-MS/MS spectra (MGF files) must be screened against a user-defined 'relevant' substance class (e.g. prenylated flavonoids) for prioritization, and you have curated 'relevant' and 'other' reference spectra to train classifiers.

## When NOT to use

- The training dataset is highly imbalanced — the article states classification of MS/MS spectra into substance classes 'most often will not be successful' in this case and the package 'is not capable of resolving this issue'
- The goal is general-purpose compound-class ontology assignment (e.g. full ClassyFire hierarchy) or molecular formula/structure elucidation rather than screening against one user-defined 'relevant' class
- The input is already a labeled feature table or classified result rather than raw MS/MS spectra needing class assignment

## Inputs

- Experimental HR-MS/MS spectra in MGF format (inference or test dataset)
- Curated training set of annotated MS/MS spectra labeled 'relevant' and 'other' (reference standards, MSnLib, in-house BOKU University databases)
- Optional: ground-truth class annotations for test spectra to compute classification accuracy

## Outputs

- Per-spectrum binary classification ('relevant' vs 'other') derived from majority vote over classifiers
- Cached MS2DeepScore embeddings on disk for reuse across runs
- Comparison table of classification fractions vs a baseline (e.g. annome_vs_sirius_comparison.csv with prenylated-flavonoid classification fractions)

## How to apply

Curate annotated training MS/MS spectra labeled 'relevant' and 'other' (e.g. from reference standards, MSnLib spectra libraries, or in-house databases) and install AnnoMe via git clone plus the uv toolkit, downloading ~11GB of public MS/MS resources with `uv run annome_downloadresources`. Compute MS2DeepScore structure-informed embeddings for the spectra — this is the limiting step (~4 hours on a standard laptop with 12 cores/16GB RAM) but is cached on disk after the first run. Train multiple classifiers (LDA, NN, SVM, etc.) with cross-validation and different random seeds, then aggregate per-spectrum results into a majority vote yielding the final 'relevant' vs 'other' label. For benchmarking, run SIRIUS 6.2.2 with ClassyFire compound-class prediction on the same test spectra, take the 'most specific class' per spectrum, and compute the fraction assigned to the target class for comparison (reported: AnnoMe ~77% correct vs SIRIUS ~41% on the prenylated-flavonoid test sets, Section 3.2, Table ST2). Use the filtering GUI (`uv run annome_filtergui`) to pre-filter MGF files and the classification GUI (`uv run annome_classificationgui`) to perform classification and comparison.

## Related tools

- **AnnoMe** (Python package providing the filtering and classification GUIs used to filter MGF files, train the LDA/NN/SVM classifiers, and derive majority-vote 'relevant'/'other' labels) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Computes the structure-informed embeddings of MS/MS spectra that serve as classifier input features; embeddings are cached on disk)
- **SIRIUS** (Baseline comparator: version 6.2.2 with ClassyFire 'most specific class' prediction run on the same test spectra to benchmark AnnoMe's classification fractions)
- **uv** (Python package/project manager used to install AnnoMe and run its resource download, filtering GUI, and classification GUI commands) — https://github.com/astral-sh/uv#installation

## Examples

```
git clone https://github.com/chrboku/AnnoMe && cd AnnoMe && uv run annome_downloadresources && uv run annome_classificationgui
```

## Evaluation signals

- Reproduced SIRIUS 6.2.2 prenylated-flavonoid fraction on the demo test dataset should match the reported ~41%; AnnoMe reference is ~77% average across the two test sets (Section 3.2, Table ST2); large deviations flag a QC failure
- Every spectrum receives exactly one final label ('relevant' or 'other') after majority-vote aggregation over multiple classifiers trained with different random seeds
- Embeddings, once computed, are cached on disk; subsequent runs on the same dataset should drop from hours to minutes
- The comparison output (e.g. annome_vs_sirius_comparison.csv) contains per-spectrum class assignments and two-column AnnoMe vs SIRIUS classification fractions

## Limitations

- Classification of MS/MS spectra into substance classes is a non-trivial task and most often will not succeed, especially with highly imbalanced training datasets; the package cannot resolve this failure mode
- Users must exercise caution with class assignments — results are a prioritization aid, not definitive structural annotations
- MS2DeepScore embedding generation is the limiting step (~4 hours on a standard laptop, Intel Core Ultra 5 125U, 16GB RAM) and could not be completed on a less powerful MacBook Air (M4, passive cooling)
- Public MS/MS resources required for setup total ~11GB and may be impractical on metered connections

## Evidence

- [abstract] AnnoMe enables user-defined class prediction: "the python-based AnnoMe software is presented that allows users to classify their experimental HR-MS/MS spectra according to their aims"
- [methods] Classifier training and majority vote: "different classifiers (LDA, NN, SVM, etc.) are first trained using cross-validation and different random seeds"
- [methods] Embeddings are the limiting step: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop"
- [readme] Embedding caching reduces runtime: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step from hours to few minutes."
- [discussion] Failure mode with imbalanced training data: "especially when the training dataset is highly imbalanced. The presented python package is not capable of resolving this issue."
- [readme] Demo workflow for prenylated flavonoids: "To execute the respective calculation, please refer to Instructions_DemoPrenylatedFlavonoids.md"
