---
name: smarts-substructure-structure-querying
description: Use when you need to convert a large public MS/MS spectral repository
  (e.g. MSnLib v6 MGF files) into labeled 'relevant'/'other' training subsets for
  a substance class of interest defined by SMARTS structural queries.
when_to_use_negative:
- Your spectra are already labeled 'relevant'/'other' (e.g. you have a curated in-house
  reference database such as the BOKU University database) — go straight to embedding
  and classification
- You want to classify individual experimental spectra rather than build a training
  set — use the AnnoMe classification GUI instead of the filtering GUI
- Your substance class of interest cannot be expressed as a SMARTS substructure query
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: AnnoMe
  role: Provides the filtering GUI used to apply SMARTS structure queries to MGF spectra
    files and label them 'relevant' or 'other'
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Python package/runner used to install and launch AnnoMe commands such as annome_downloadresources
    and annome_filtergui
  repo: https://github.com/astral-sh/uv
- name: git
  role: Used to obtain a copy of the AnnoMe repository before running the filtering
    GUI
  repo: https://github.com/git-guides/install-git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:fc1b0e4442dabc661a8eaad4ab2926d867c5fdd12f039ea33a8d6e20782f3591
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - Python
  - git
  - uv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# smarts-substructure-structure-querying

## Summary

Filter large public MSnLib MGF spectra libraries with user-defined SMARTS structure queries using the AnnoMe filtering GUI, labeling matched spectra as 'relevant' and the remainder as 'other' to build class-specific training sets for HR-MS/MS spectrum classifiers.

## When to use

Use when you need to convert a large public MS/MS spectral repository (e.g. MSnLib v6 MGF files) into labeled 'relevant'/'other' training subsets for a substance class of interest defined by SMARTS structural queries.

## When NOT to use

- Your spectra are already labeled 'relevant'/'other' (e.g. you have a curated in-house reference database such as the BOKU University database) — go straight to embedding and classification
- You want to classify individual experimental spectra rather than build a training set — use the AnnoMe classification GUI instead of the filtering GUI
- Your substance class of interest cannot be expressed as a SMARTS substructure query

## Inputs

- Public MS/MS resources (~11 GB) including MSnLib v6 MGF spectra files with associated compound structures
- SMARTS structure queries defining the substance class of interest (e.g. (iso-)flavonoids)
- Spectral subset definitions by polarity and collision energy

## Outputs

- Filtered and labeled MGF spectra subsets ('relevant' vs 'other') per polarity and collision-energy subset
- Labeled training corpus suitable for MS2DeepScore embedding and classifier training

## How to apply

Download the public MS/MS resources with `uv run annome_downloadresources` (~11 GB, resumable), then launch the AnnoMe filtering GUI (`uv run annome_filtergui`). Define SMARTS queries encoding the structural motif of interest (e.g. (iso-)flavonoid scaffolds) and apply them to the MGF spectra structure associations, labeling spectra as 'relevant' or 'other'. Perform the filtering separately for each polarity and collision-energy spectral subset, since classifier training in the AnnoMe workflow is done per subset. The output labeled spectra subsets then feed into MS2DeepScore embedding computation and LDA/NN/SVM classifier training via the classification GUI, so filter choices directly determine the class balance and quality of the downstream models.

## Related tools

- **AnnoMe** (Provides the filtering GUI used to apply SMARTS structure queries to MGF spectra files and label them 'relevant' or 'other') — https://github.com/chrboku/AnnoMe
- **uv** (Python package/runner used to install and launch AnnoMe commands such as annome_downloadresources and annome_filtergui) — https://github.com/astral-sh/uv
- **git** (Used to obtain a copy of the AnnoMe repository before running the filtering GUI) — https://github.com/git-guides/install-git

## Examples

```
uv run annome_downloadresources && uv run annome_filtergui
```

## Evaluation signals

- Each polarity and collision-energy subset has separate 'relevant' and 'other' labels assigned after filtering
- The filtered spectra counts are consistent with SMARTS matches against the compound structures associated with the MSnLib MGF entries
- Downstream training on the filtered subsets reproduces reported metrics (balanced accuracy 0.866 training / 0.82 test; TPR >55% training / 82% test; FPR <10% training / 5.2% test)
- The full resource download completed (~11 GB) and the resumable download terminated without missing MGF files before filtering starts

## Limitations

- All public MS/MS resources total approximately 11 GB (per 2.2026), so the download can be slow and must be metered-connection aware, though it is interruptible and resumable
- Classification of MS/MS spectra into substance classes is a non-trivial task and most often will not be successful, especially with highly imbalanced training datasets; AnnoMe cannot resolve this, so results must be treated with caution
- Filtering quality directly bounds downstream classifier performance since labels derive from SMARTS matching rather than experimental reference standards

## Evidence

- [readme] filtering GUI purpose: "AnnoMe provides two graphical user interface programs for a) the filtering of MGF files for interesting structures, and b) performing classification and comparison tasks."
- [methods] filtering GUI command: "Start the filtering GUI via the command `uv run annome_filtergui`"
- [readme] download resources command and size: "Download available public MS/MS resources using the command `uv run annome_downloadresources` or double-click the file `annome_downloadresources.bat` (Windows)"
- [intro] relevant/other labeling purpose: "This is a package for the classification of MS/MS spectra of novel compounds into either "relevant" or "other" compounds."
- [readme] classifiers trained on filtered labeled spectra: "Using training data consisting of annotated and labeled MS/MS spectra of compounds of interest and others, different classifiers (LDA, NN, SVM, etc.) are first trained"
