---
name: msms-spectral-library-filtering
description: 'Use when building training data for MS/MS spectrum classification: you
  have MGF spectral library files and SMARTS structure queries for a compound class
  of interest and need to label spectra as ''relevant'' or ''other''.'
when_to_use_negative:
- When the goal is direct spectral library search/annotation of unknown spectra rather
  than assembling labeled training data for a custom classifier
- When the training dataset is highly imbalanced — the article explicitly warns that
  classification will most often not succeed in that case and AnnoMe cannot resolve
  this issue
- When your compound class cannot be expressed as a SMARTS structure query over library
  structures
edam_operation: http://edamontology.org/operation_3760
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Provides the filtering GUI (`uv run annome_filtergui`) used to filter MGF
    library files by SMARTS structure queries and label spectra as 'relevant' or 'other'
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Python package/runner used to install and launch all AnnoMe commands (`uv
    run annome_downloadresources`, `annome_filtergui`, `annome_classificationgui`)
  repo: https://github.com/astral-sh/uv
- name: git
  role: Used to obtain a copy of the AnnoMe repository before running the filtering
    workflow
  repo: https://github.com/git-guides/install-git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:0513c9cd447c82ddbbf4f0642d787739da8dd6514dbb4fc99e18561419a242eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - Python
  - git
  - uv
  edam_operation: http://edamontology.org/operation_3760
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# msms-spectral-library-filtering

## Summary

Filter large public HR-MS/MS spectral libraries (MGF format, e.g. MSnLib v6) down to spectra of a user-defined substance class by applying SMARTS structure queries to library structures, using the AnnoMe filtering GUI. This creates labeled 'relevant'/'other' training sets for spectral classification, split per polarity and collision-energy subset.

## When to use

Use when building training data for MS/MS spectrum classification: you have MGF spectral library files and SMARTS structure queries for a compound class of interest and need to label spectra as 'relevant' or 'other'.

## When NOT to use

- When the goal is direct spectral library search/annotation of unknown spectra rather than assembling labeled training data for a custom classifier
- When the training dataset is highly imbalanced — the article explicitly warns that classification will most often not succeed in that case and AnnoMe cannot resolve this issue
- When your compound class cannot be expressed as a SMARTS structure query over library structures

## Inputs

- MSnLib v6 MGF spectra library files (downloaded via `uv run annome_downloadresources`, ~11 GB total)
- SMARTS structure queries defining the substance class of interest (e.g. (iso-)flavonoids)
- Library compound structure records associated with each MGF spectrum

## Outputs

- Filtered MGF spectral subsets labeled 'relevant' or 'other', split per polarity and per collision-energy condition
- Labeled training sets suitable for MS2DeepScore embedding and classifier training (LDA, NN, SVM)

## How to apply

First download the public MS/MS resources (~11 GB, resumable) with `uv run annome_downloadresources`. Then start the AnnoMe filtering GUI (`uv run annome_filtergui`) and filter the MSnLib v6 MGF spectra files using SMARTS structure queries against the library compound structures, labeling matched spectra as 'relevant' and the remainder as 'other' separately for each polarity and collision-energy subset. The per-subset separation matters because spectra acquired under different polarity and collision-energy conditions are classified by separate models. The filtered, labeled subsets then feed the downstream MS2DeepScore embedding and classifier-training workflow (LDA, NN, SVM with cross-validation and majority vote) via `uv run annome_classificationgui`.

## Related tools

- **AnnoMe** (Provides the filtering GUI (`uv run annome_filtergui`) used to filter MGF library files by SMARTS structure queries and label spectra as 'relevant' or 'other') — https://github.com/chrboku/AnnoMe
- **uv** (Python package/runner used to install and launch all AnnoMe commands (`uv run annome_downloadresources`, `annome_filtergui`, `annome_classificationgui`)) — https://github.com/astral-sh/uv
- **git** (Used to obtain a copy of the AnnoMe repository before running the filtering workflow) — https://github.com/git-guides/install-git

## Examples

```
uv run annome_downloadresources && uv run annome_filtergui
```

## Evaluation signals

- The filtering GUI runs without error and produces per-polarity and per-collision-energy subsets from the MSnLib v6 MGF files
- Every spectrum in the output subsets carries a 'relevant' or 'other' label traceable to a SMARTS query match against its library structure
- Downstream, classifiers trained on the filtered subsets achieve balanced accuracy in the reported range (~0.866 sd 3% on training data; ~0.82 sd 4% on the independent test set, min 0.76, max 0.87)
- Reported evaluation metrics on the test set match Table ST1 expectations: TPR ~82% and FPR ~5.2%
- The 11 GB resource download can be interrupted and restarted without corruption (resumable download check)

## Limitations

- Classification of MS/MS spectra into substance classes is non-trivial and will most often fail, especially with highly imbalanced training datasets; AnnoMe cannot resolve this
- Users are warned to exercise caution with the assignment of MS/MS spectra to specific compound classes of interest
- Public MS/MS resources total approximately 11 GB (as of 2.2026); download size matters on metered connections, though the download is resumable

## Evidence

- [methods] Start the filtering GUI via the command `uv run annome_filtergui`: "Start the filtering GUI via the command `uv run annome_filtergui`"
- [methods] Download available public MS/MS resources using the command `uv run annome_downloadresources`: "Download available public MS/MS resources using the command `uv run annome_downloadresources`"
- [abstract] By leveraging a user-curated dataset of "relevant" and "other" reference HR-MS/MS spectra: "By leveraging a user-curated dataset of "relevant" and "other" reference HR-MS/MS spectra"
- [readme] All resources are approximately 11GB in size (per 2.2026), and - depending on your internet connection - the download might take some time. It can be interrupted and restarted.: "All resources are approximately 11GB in size (per 2.2026), and - depending on your internet connection - the download might take some time. It can be interrupted and restarted."
- [methods] MSnLib spectra libraries: "MSnLib spectra libraries"
