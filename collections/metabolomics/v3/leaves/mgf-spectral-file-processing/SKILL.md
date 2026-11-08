---
name: mgf-spectral-file-processing
description: Use when HR-MS/MS inference spectra or spectral libraries are supplied
  as MGF files and must be loaded, SMARTS-filtered into 'relevant'/'other' sets, or
  exported before classifier training/prediction in AnnoMe.
when_to_use_negative:
- Input spectra are in mzML or another vendor/centroided format rather than MGF, since
  AnnoMe's filtering and classification GUIs operate on MGF datasets
- The training dataset is highly imbalanced and the goal is reliable substance-class
  assignment — the article states classification will most often fail in that case
  and the package cannot resolve it
- You need de novo structure elucidation or compound identification rather than binary
  'relevant'/'other' classification of spectra
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Python package providing annome_filtergui and annome_classificationgui for
    loading, SMARTS-filtering, and classifying MGF spectra
  repo: https://github.com/chrboku/AnnoMe
- name: MS2DeepScore
  role: Computes structure-informed embeddings for MGF spectra; embeddings are cached
    on disk after first computation
- name: uv
  role: Toolkit used to run AnnoMe entry points (annome_downloadresources, annome_filtergui,
    annome_classificationgui)
  repo: https://github.com/astral-sh/uv
- name: git
  role: Used to obtain a copy of the AnnoMe repository
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:00d69c54f8a5855d3c497aa5f97c359f016ec7ea5d6f10877b3d9d5407be28c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - MS2DeepScore
  - Python
  - git
  - uv
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# mgf-spectral-file-processing

## Summary

Loading, filtering, and exporting HR-MS/MS spectra in MGF format with the AnnoMe package, so that spectral libraries (e.g. MSnLib v6) and inference datasets (plant extracts) can be prepared for 'relevant'/'other' classification.

## When to use

Use when HR-MS/MS inference spectra or spectral libraries are supplied as MGF files and must be loaded, SMARTS-filtered into 'relevant'/'other' sets, or exported before classifier training/prediction in AnnoMe.

## When NOT to use

- Input spectra are in mzML or another vendor/centroided format rather than MGF, since AnnoMe's filtering and classification GUIs operate on MGF datasets
- The training dataset is highly imbalanced and the goal is reliable substance-class assignment — the article states classification will most often fail in that case and the package cannot resolve it
- You need de novo structure elucidation or compound identification rather than binary 'relevant'/'other' classification of spectra

## Inputs

- MGF inference spectra files (e.g. Glycyrrhiza uralensis root and Paulownia tomentosa fruit extract HR-MS/MS spectra)
- MGF spectral library (MSnLib v6)
- SMARTS substructure query strings for (iso-)flavonoids, prenylated flavonoids, and prenylated chalcones
- Pre-trained AnnoMe classifiers (LDA, NN, SVM) with per-spectrum majority-vote aggregation

## Outputs

- Filtered MGF files with 'relevant'/'other' labels
- Class-count summary table (e.g. 655 relevant / 10 000 other for (iso-)flavonoids on MSnLib v6)
- Per-spectrum classification labels and per-sample counts (e.g. classification_results.csv)

## How to apply

Obtain AnnoMe via `git clone https://github.com/chrboku/AnnoMe` (or a versioned release) and install the uv toolkit, then run `uv run annome_downloadresources` to fetch public MS/MS resources (~11 GB, resumable/incremental). Launch the filtering GUI with `uv run annome_filtergui` and load the input MGF dataset, e.g. the MSnLib v6 spectral library, applying (iso-)flavonoid, prenylated-flavonoid, or prenylated-chalcone SMARTS substructure queries so matching structures are labeled 'relevant' and the remainder 'other'. Export the filtered spectra as MGF files and compile class counts — the (iso-)flavonoid filter on MSnLib v6 should yield exactly 655 'relevant' and 10 000 'other' spectra. For inference, load the deposited MGF spectra of Glycyrrhiza uralensis root and Paulownia tomentosa fruit extracts into the classification GUI (`uv run annome_classificationgui`); MS2DeepScore embeddings are computed once and cached on disk, reducing later runs from hours to minutes.

## Related tools

- **AnnoMe** (Python package providing annome_filtergui and annome_classificationgui for loading, SMARTS-filtering, and classifying MGF spectra) — https://github.com/chrboku/AnnoMe
- **MS2DeepScore** (Computes structure-informed embeddings for MGF spectra; embeddings are cached on disk after first computation)
- **uv** (Toolkit used to run AnnoMe entry points (annome_downloadresources, annome_filtergui, annome_classificationgui)) — https://github.com/astral-sh/uv
- **git** (Used to obtain a copy of the AnnoMe repository)

## Examples

```
git clone https://github.com/chrboku/AnnoMe; cd AnnoMe; uv run annome_downloadresources; uv run annome_filtergui
```

## Evaluation signals

- The (iso-)flavonoid SMARTS filter on MSnLib v6 yields exactly 655 'relevant' and 10 000 'other' spectra, matching Section 3.1
- Per-sample prenylated-flavonoid counts fall in the reported ranges: Glycyrrhiza uralensis 231 (25%) and 305 (15%) spectra; Paulownia tomentosa 171–1005 spectra (17%–34%)
- Downloaded public MS/MS resources total approximately 11 GB and the download can be interrupted and restarted
- MS2DeepScore embeddings are cached on disk after the first run, cutting subsequent embedding runtime from hours to a few minutes
- Exported filtered datasets are valid MGF files that reload correctly in annome_classificationgui

## Limitations

- Generation of MS2DeepScore embeddings took ~4 hours on a standard laptop (Intel Core Ultra 5 125U, 12 cores; 16GB RAM) and the same dataset could not be processed on a less powerful MacBook Air (M4, 16GB RAM, passive cooling)
- Classification of MS/MS spectra into substance classes is non-trivial and most often will not succeed, especially with highly imbalanced training datasets; the package cannot resolve this
- All public MS/MS resources total ~11 GB, so bandwidth and disk space must be considered; no changelog was found for the repository

## Evidence

- [methods] filtering GUI usage: "Start the filtering GUI via the command `uv run annome_filtergui`"
- [methods] classification GUI usage: "Start the classification GUI via the command `uv run annome_classificationgui`"
- [methods] resource download: "Download available public MS/MS resources using the command `uv run annome_downloadresources`"
- [methods] embedding caching: "Once these have been calculated, the embeddings are cached on disk, which significantly reduces the duration of this step"
- [readme] MGF classification GUI purpose: "AnnoMe provides two graphical user interface programs for a) the filtering of MGF files for interesting structures, and b) performing classification and comparison tasks."
- [readme] resource size caveat: "Note: All resources are approximately 11GB in size (per 2.2026), and - depending on your internet connection - the download might take some time. It can be interrupted and restarted."
