---
name: classifier-prediction-count-validation
description: Use when an article reports integer counts or fractions of MS/MS spectra
  classified into a compound class and you must reproduce those tallies from deposited
  MGF inference spectra plus trained classifiers.
when_to_use_negative:
- You need to assign individual spectra to a compound class where the training dataset
  is highly imbalanced — the article explicitly warns the package cannot resolve this
  failure mode
- You require definitive structural annotation rather than prioritization-level classification;
  AnnoMe yields 'relevant'/'other' flags, not identity assignments
- Only a few spectra must be processed and embedding generation on weak hardware (e.g.
  passively cooled MacBook Air) is impractical
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: AnnoMe
  role: Runs MS2DeepScore embedding, per-classifier prediction and majority-vote aggregation
    on MGF inference spectra, producing the 'relevant'/'other' labels to be tallied
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Computes the structure-informed spectral embeddings that feed the trained
    classifiers; cached on disk after first computation
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Execution environment used to launch the AnnoMe classification GUI and resource
    download commands
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:271fdead2b0d9a88b473457b519a6083033d72594d40a62973113953335b7b28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - uv
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Classifier prediction count validation

## Summary

Reproduces and validates the per-sample counts of HR-MS/MS spectra that AnnoMe majority-vote classifiers label as a substance class of interest (e.g. prenylated flavonoids) in plant-extract inference datasets such as Glycyrrhiza uralensis root and Paulownia tomentosa fruit extracts.

## When to use

Use when an article reports integer counts or fractions of MS/MS spectra classified into a compound class and you must reproduce those tallies from deposited MGF inference spectra plus trained classifiers.

## When NOT to use

- You need to assign individual spectra to a compound class where the training dataset is highly imbalanced — the article explicitly warns the package cannot resolve this failure mode
- You require definitive structural annotation rather than prioritization-level classification; AnnoMe yields 'relevant'/'other' flags, not identity assignments
- Only a few spectra must be processed and embedding generation on weak hardware (e.g. passively cooled MacBook Air) is impractical

## Inputs

- Deposited MGF inference spectra from plant extracts (Glycyrrhiza uralensis root, Paulownia tomentosa fruit)
- Pre-trained AnnoMe classifiers for the target substance class (LDA, NN, SVM etc. trained with cross-validation and different random seeds)
- Public MS/MS resources downloaded via annome_downloadresources (~11 GB, incl. MSnLib libraries)

## Outputs

- Per-spectrum majority-vote labels ('relevant' vs 'other') for every HR-MS/MS spectrum
- Per-sample counts and fractions of spectra classified as the target class (e.g. 231 and 305 for Glycyrrhiza uralensis; 171–1005 for Paulownia tomentosa)
- classification_results.csv with tallies comparable to article Table ST4

## How to apply

Load the deposited MGF inference spectra (e.g. Glycyrrhiza uralensis root and Paulownia tomentosa fruit extracts) and the pre-trained classifiers (LDA, NN, SVM, etc.) into AnnoMe. Compute MS2DeepScore structure-informed embeddings for all inference spectra; the first embedding pass takes ~4 hours on a 12-core laptop and results are cached on disk so subsequent runs take minutes. Apply each individual trained classifier to the embeddings and aggregate per-spectrum predictions by majority vote to derive the final 'relevant'/'other' label for every HR-MS/MS spectrum. Tally the number and fraction of 'relevant' spectra per plant sample and compare against the reported benchmark (Glycyrrhiza uralensis: 231 (25%) and 305 (15%) spectra in positive/negative ionization modes; Paulownia tomentosa: 171–1005 spectra, 17%–34%). Report per-spectrum labels and per-sample counts, treating the comparison table as the acceptance criterion rather than re-training classifiers.

## Related tools

- **AnnoMe** (Runs MS2DeepScore embedding, per-classifier prediction and majority-vote aggregation on MGF inference spectra, producing the 'relevant'/'other' labels to be tallied) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Computes the structure-informed spectral embeddings that feed the trained classifiers; cached on disk after first computation) — https://github.com/chrboku/AnnoMe
- **uv** (Execution environment used to launch the AnnoMe classification GUI and resource download commands) — https://github.com/astral-sh/uv

## Examples

```
uv run annome_classificationgui
```

## Evaluation signals

- Glycyrrhiza uralensis counts reproduce the reported 231 (25%) and 305 (15%) spectra across the two ionization polarities
- Paulownia tomentosa counts fall in the reported 171–1005 range with 17%–34% classified as prenylated flavonoids
- Per-spectrum labels are binary ('relevant'/'other') and every spectrum in the MGF input receives exactly one aggregated label
- Embedding cache exists after the first run, reducing subsequent embedding time from hours to a few minutes
- Per-sample tallies match Table ST4 of the article when the same deposited MGF and classifiers are used

## Limitations

- Classification of MS/MS spectra into substance classes most often fails when the training dataset is highly imbalanced; the package cannot resolve this
- Users are explicitly warned to exercise caution with assignments of MS/MS spectra to specific compound classes
- Embedding generation is the time-limiting step (~4 h on a 12-core laptop) and failed entirely on a less powerful passively-cooled MacBook Air (M4, 16 GB RAM)
- Results depend on the majority vote of classifiers trained with cross-validation and different random seeds; no per-classifier performance is surfaced by the aggregation

## Evidence

- [results] Glycyrrhiza uralensis counts reproduce the reported 231 (25%) and 305 (15%) spectra: [evidence span withheld — anchor status: not_found]
- [results] Paulownia counts fall in the reported range: [evidence span withheld — anchor status: not_found]
- [readme] Embedding is the bottleneck and is cached: "The limiting step is the generation of the MS2DeepScore embeddings, which took 4 hours on a standard laptop"
- [readme] Majority-vote aggregation defines the final label: "The results are then aggregated and a majority-vote is derived which indicates the final classification of the input MS/MS spectra to either "relevant" or "other""
- [readme] Imbalanced training data is a known failure mode: "Classification of MS/MS spectra into substance classes is a non-trivial task and most often the classification will not be successful"
- [readme] Command-line entry point for classification: "uv run annome_classificationgui"
