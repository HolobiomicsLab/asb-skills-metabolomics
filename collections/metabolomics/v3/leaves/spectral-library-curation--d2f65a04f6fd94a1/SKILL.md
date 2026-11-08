---
name: spectral-library-curation
description: Use when building labeled MS/MS training data for spectrum classification,
  e.g. splitting an MGF spectral library such as MSnLib v6 into 'relevant' vs 'other'
  classes using SMARTS substructure queries.
when_to_use_negative:
- Spectra lack associated compound structures, so SMARTS substructure filtering cannot
  assign 'relevant'/'other' labels
- The resulting training dataset is highly imbalanced — the README warns classification
  'will not be successful' in that case and AnnoMe cannot resolve it
- Reference spectra are already labeled (e.g. from reference standards or the BOKU
  in-house database); filtering would be redundant
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnoMe
  role: Provides the annome_filtergui used to load MGF libraries, apply SMARTS substructure
    queries, and export labeled 'relevant'/'other' MGF files
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Toolkit used to install and run the AnnoMe entry points (annome_downloadresources,
    annome_filtergui)
  repo: https://github.com/astral-sh/uv
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:098cfb28fae9f02b9fd25e8e75dc14c43f234d046c201ab4e020bdc3b866376f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chrboku/AnnoMe
  tools:
  - AnnoMe
  - Python
  - uv
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-library-curation

## Summary

Curates labeled HR-MS/MS training libraries by downloading public spectral resources and applying SMARTS substructure filtering to split spectra into user-defined 'relevant' and 'other' classes, as implemented in the AnnoMe filtering GUI on the MSnLib v6 library.

## When to use

Use when building labeled MS/MS training data for spectrum classification, e.g. splitting an MGF spectral library such as MSnLib v6 into 'relevant' vs 'other' classes using SMARTS substructure queries.

## When NOT to use

- Spectra lack associated compound structures, so SMARTS substructure filtering cannot assign 'relevant'/'other' labels
- The resulting training dataset is highly imbalanced — the README warns classification 'will not be successful' in that case and AnnoMe cannot resolve it
- Reference spectra are already labeled (e.g. from reference standards or the BOKU in-house database); filtering would be redundant

## Inputs

- MGF-format MS/MS spectral library (e.g. MSnLib v6) with structure annotations
- SMARTS substructure query strings defining the 'relevant' class (e.g. (iso)-flavonoid, prenylated flavonoid, prenylated chalcone)
- AnnoMe public MS/MS resources (~11 GB, downloaded via annome_downloadresources)

## Outputs

- Labeled 'relevant' and 'other' MGF files of filtered spectra
- Class-count summary table (expected: 655 (iso)-flavonoid 'relevant' vs 10000 'other' spectra for MSnLib v6)

## How to apply

Install the uv toolkit and obtain AnnoMe via 'git clone https://github.com/chrboku/AnnoMe', then fetch the public MS/MS resources with 'uv run annome_downloadresources' (about 11 GB per 2.2026, resumable if interrupted). Launch the filtering GUI with 'uv run annome_filtergui' and load the MSnLib v6 MGF spectral library as the input dataset. Apply the (iso)-flavonoid SMARTS substructure queries so matching structures are labeled 'relevant' and remaining structures 'other', reproducing the 655 relevant versus 10000 other split, then repeat with prenylated-flavonoid and prenylated-chalcone substructure definitions for a second split. Export the filtered spectra as MGF files and compile the class counts into a summary table; these curated, labeled libraries then feed classifier training (LDA, NN, SVM) on MS2DeepScore embeddings with majority-vote aggregation.

## Related tools

- **AnnoMe** (Provides the annome_filtergui used to load MGF libraries, apply SMARTS substructure queries, and export labeled 'relevant'/'other' MGF files) — https://github.com/chrboku/AnnoMe
- **uv** (Toolkit used to install and run the AnnoMe entry points (annome_downloadresources, annome_filtergui)) — https://github.com/astral-sh/uv

## Examples

```
uv run annome_filtergui
```

## Evaluation signals

- The (iso)-flavonoid SMARTS filter on MSnLib v6 yields exactly 655 'relevant' and 10000 'other' spectra, matching the Section 3.1 counts
- Exported MGF files parse correctly and every spectrum entry carries its 'relevant' or 'other' label
- Downstream cross-validated classifiers trained on the curated library reach specificity of at least 99.4% and average sensitivity of about 75.7%
- The resource download completes without corruption and can be interrupted/restarted incrementally

## Limitations

- Classification of MS/MS spectra into substance classes most often fails, especially with highly imbalanced training datasets, and AnnoMe cannot resolve this; users are told to exercise caution with class assignments
- Public MS/MS resources total roughly 11 GB and depend on connection bandwidth, though the download is resumable
- Downstream MS2DeepScore embedding generation took 4 hours on a standard laptop (12 cores, 16 GB RAM) and could not be run on a less powerful fanless MacBook Air

## Evidence

- [readme] classification of MS/MS spectra into relevant/other classes: "This is a package for the classification of MS/MS spectra of novel compounds into either "relevant" or "other" compounds."
- [readme] resource download entry point: "Download available public MS/MS resources using the command `uv run annome_downloadresources` or double-click the file `annome_downloadresources.bat` (Windows)"
- [methods] filtering GUI launch: "Start the filtering GUI via the command `uv run annome_filtergui`"
- [abstract] majority-vote classification outcome: "the package enables rapid and accurate prediction of "relevant" compounds with custom-trained classification models and a majority vote"
