---
name: smarts-substructure-filtering
description: Use when training-set curation for spectral classification needs labeled
  'relevant'/'other' classes, and structures in an MGF spectral library can be separated
  by SMARTS substructure definitions of a substance class.
when_to_use_negative:
- When you already have a curated, labeled set of 'relevant' and 'other' reference
  MS/MS spectra (e.g., from reference standards) and do not need structure-based filtering
  to generate it.
- When the target training dataset is highly imbalanced — the article notes classification
  often fails in this case and AnnoMe cannot resolve it.
- When the structures behind the spectra are unknown or cannot be matched by SMARTS
  queries, since filtering relies on substructure matching of library structures.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0605
tools:
- name: AnnoMe
  role: Provides the annome_filtergui used to apply SMARTS substructure queries to
    an MGF spectral library and label spectra as 'relevant'/'other'; also provides
    annome_downloadresources to fetch the public MS/MS resources.
  repo: https://github.com/chrboku/AnnoMe
- name: uv
  role: Python package/runner toolkit used to install and launch AnnoMe entry points
    (e.g., `uv run annome_filtergui`).
  repo: https://github.com/astral-sh/uv
- name: git
  role: Used to obtain a copy of the AnnoMe repository (`git clone https://github.com/chrboku/AnnoMe`).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag111
    title: 'AnnoMe: user-defined classification of HR-MS/MS spectra for natural product
      discovery'
  source_content_hash: sha256:1fc47654af0bea60f1593ee87d6abb19b149f509469c00d398b964fe71b69922
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
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# smarts-substructure-filtering

## Summary

Build user-defined 'relevant'/'other' training sets for MS/MS spectral classification by applying SMARTS substructure queries to the structures behind an MGF spectral library, using AnnoMe's filtering GUI. This step produced the 655 (iso-)flavonoid 'relevant' vs 10 000 'other' spectra used to train AnnoMe classifiers on MSnLib v6.

## When to use

Use when training-set curation for spectral classification needs labeled 'relevant'/'other' classes, and structures in an MGF spectral library can be separated by SMARTS substructure definitions of a substance class.

## When NOT to use

- When you already have a curated, labeled set of 'relevant' and 'other' reference MS/MS spectra (e.g., from reference standards) and do not need structure-based filtering to generate it.
- When the target training dataset is highly imbalanced — the article notes classification often fails in this case and AnnoMe cannot resolve it.
- When the structures behind the spectra are unknown or cannot be matched by SMARTS queries, since filtering relies on substructure matching of library structures.

## Inputs

- MGF spectral library file (e.g., MSnLib v6) with structures annotated for each spectrum
- SMARTS substructure query strings defining the 'relevant' substance class (e.g., (iso-)flavonoid, prenylated-flavonoid, prenylated-chalcone SMARTS)
- AnnoMe package installed via git clone https://github.com/chrboku/AnnoMe with the uv toolkit

## Outputs

- Filtered MGF files with spectra labeled 'relevant' (substructure matches) and 'other' (non-matching)
- Class counts summary (e.g., 655 relevant vs 10 000 other spectra for (iso-)flavonoids on MSnLib v6)

## How to apply

Install AnnoMe via git clone (or a versioned release) and the uv toolkit, then fetch public MS/MS resources with `uv run annome_downloadresources` (~11 GB, resumable/incremental). Launch the filtering GUI with `uv run annome_filtergui` and load the MGF spectral library of interest (e.g., MSnLib v6) as input. Apply the SMARTS substructure queries defining the substance class of interest — here (iso-)flavonoid SMARTS strings — so that structures matching the queries are labeled 'relevant' and all remaining structures are labeled 'other'; repeat with class-specific definitions such as prenylated-flavonoid and prenylated-chalcone SMARTS for narrower classes. Export the filtered spectra as MGF files and compile the class counts, which should be checked against reported reference counts (655 'relevant' vs 10 000 'other' for (iso-)flavonoids on MSnLib v6). These labeled MGF files then serve as the training data for AnnoMe's classifiers (LDA, NN, SVM with cross-validation and majority vote) using MS2DeepScore embeddings.

## Related tools

- **AnnoMe** (Provides the annome_filtergui used to apply SMARTS substructure queries to an MGF spectral library and label spectra as 'relevant'/'other'; also provides annome_downloadresources to fetch the public MS/MS resources.) — https://github.com/chrboku/AnnoMe
- **uv** (Python package/runner toolkit used to install and launch AnnoMe entry points (e.g., `uv run annome_filtergui`).) — https://github.com/astral-sh/uv
- **git** (Used to obtain a copy of the AnnoMe repository (`git clone https://github.com/chrboku/AnnoMe`).)

## Examples

```
git clone https://github.com/chrboku/AnnoMe; cd AnnoMe; uv run annome_downloadresources; uv run annome_filtergui
```

## Evaluation signals

- The (iso-)flavonoid SMARTS filter on MSnLib v6 yields exactly 655 'relevant' spectra and 10 000 'other' spectra, matching Section 3.1 of the paper.
- Every spectrum labeled 'relevant' contains a structure that matches at least one of the applied SMARTS queries; no 'other' spectrum matches the 'relevant' SMARTS definitions.
- Filtered spectra export as valid MGF files loadable back into the AnnoMe classification GUI for downstream classifier training.
- A second filtering pass with prenylated-flavonoid and prenylated-chalcone SMARTS reproduces the Section 3.2 'relevant'/'other' split.

## Limitations

- All public MS/MS resources total approximately 11 GB (per 2.2026); the download can be interrupted and restarted but may take considerable time, which matters on metered connections.
- Class label quality depends entirely on the SMARTS definitions chosen; the paper's demonstration covers (iso-)flavonoids and prenylated (iso)flavonoids/chalcones, and other classes require user-supplied SMARTS.
- The labeled training data produced by this filtering are subject to the downstream warning that spectral classification into substance classes is non-trivial and often unsuccessful, especially with highly imbalanced training datasets.

## Evidence

- [methods] filtering of MGF files using the filtering GUI: "Start the filtering GUI via the command `uv run annome_filtergui`"
- [methods] resource download entry point: "Download available public MS/MS resources using the command `uv run annome_downloadresources`"
- [readme] resource size and resumability: "All resources are approximately 11GB in size (per 2.2026), and - depending on your internet connection - the download might take some time."
- [readme] demonstration classes covered: "There are two demonstrations available in AnnoMe. These try to train classifiers for - Flavonoid and isoflavonoid compounds - Prenylated flavonoids and isoflavonoids and prenylated chalcones."
- [intro] downstream classifier training context: "the classifiers first need to be trained on a large set of MS/MS spectra of compounds of interest (e.g., obtained from reference standards) and others"
- [discussion] imbalance limitation: "especially when the training dataset is highly imbalanced. The presented python package is not capable of resolving this issue."
