---
name: wheat-metabolomics-feature-handling
description: Use when a set of MS/MS feature spectra in .mgf format needs a 2D similarity-based
  overview embedding, or when reproducing the paper's wheat-data t-SNE grouping of
  features around reference standards like Procyanidin-A2.
when_to_use_negative:
- Datasets larger than ~5000 features — the article anticipates the dashboard will
  face difficulties for larger data sets and liberal settings
- Goal is library annotation or analog search (MS2Query library search) rather than
  similarity-matrix embedding and local connectivity exploration
- Only a static overview is needed and no expert visual follow-up is possible — the
  t-SNE overview figure is difficult to read on its own
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: matchms
  role: Imports the .mgf MS/MS spectral data into Python and allows adding/renaming
    the 'feature_id' metadata key on existing spectra
  repo: https://matchms.readthedocs.io/en/latest/
- name: ms2deepscore
  role: Provides the pretrained machine-learning pairwise similarity scores that form
    the quantitative backbone of the t-SNE embedding
  repo: https://github.com/matchms/ms2deepscore
- name: MS2Query
  role: Supplies the pretrained model files (positive/negative mode) required by the
    importing pipeline to compute ms2deepscore scores
- name: specXplore
  role: Importing pipeline and interactive dashboard that wraps the t-SNE embedding
    of the ms2deepscore matrix with network views, heatmaps, and fragmentation overview
    maps
  repo: https://github.com/kevinmildau/specXplore
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:12f046f6c916137a9df93346ee553d18d75121c12dbccf383d0f8f46f688392e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# wheat-metabolomics-feature-handling

## Summary

Reproduce the two-dimensional t-SNE overview embedding of pairwise ms2deepscore similarities for LC-MS/MS feature data (e.g. the wheat dataset), as used by the specXplore dashboard as its quantitative backbone. This yields a per-feature 2D layout in which high-similarity spectral features group locally, e.g. around the Procyanidin-A2 reference standard.

## When to use

Use when a set of MS/MS feature spectra in .mgf format needs a 2D similarity-based overview embedding, or when reproducing the paper's wheat-data t-SNE grouping of features around reference standards like Procyanidin-A2.

## When NOT to use

- Datasets larger than ~5000 features — the article anticipates the dashboard will face difficulties for larger data sets and liberal settings
- Goal is library annotation or analog search (MS2Query library search) rather than similarity-matrix embedding and local connectivity exploration
- Only a static overview is needed and no expert visual follow-up is possible — the t-SNE overview figure is difficult to read on its own

## Inputs

- .mgf file with MS/MS feature spectra (unique feature identifier with key 'feature_id', precursor m/z, m/z-intensity tuples)
- MS2Query pretrained model and library files (positive mode: zenodo.org/records/10527997; negative mode: zenodo.org/records/10528030)
- Optionally a precomputed ms2deepscore pairwise similarity matrix

## Outputs

- tsne_embedding.csv with one row per feature and x/y columns (2D t-SNE coordinates of the ms2deepscore similarity matrix)
- A specXplore session data object consumable by the interactive dashboard

## How to apply

Import the .mgf MS/MS feature spectra into Python with matchms, ensuring each spectrum carries the feature identifier key 'feature_id' (specXplore expects this key; it can be added via matchms.Spectrum metadata or renamed in a text editor). Compute (or load) the pairwise ms2deepscore similarity matrix for all features using the pretrained models provided with MS2Query (positive- and negative-mode model/library files from Zenodo, kept in separate 'pos'/'neg' folders). Run t-SNE (scikit-learn) on the ms2deepscore pairwise similarity matrix, which serves as the primary similarity score; test perplexity values in the usual range of 5–50 and select an appropriate value balancing high-dimensional distance preservation against qualitative grouping. Save the final embedding as tsne_embedding.csv with one row per feature containing x and y columns. Validate that the file has exactly one coordinate row per wheat feature and that expert visual review confirms high-similarity features group around the Procyanidin-A2 feature, consistent with the paper's illustrative wheat example.

## Related tools

- **matchms** (Imports the .mgf MS/MS spectral data into Python and allows adding/renaming the 'feature_id' metadata key on existing spectra) — https://matchms.readthedocs.io/en/latest/
- **ms2deepscore** (Provides the pretrained machine-learning pairwise similarity scores that form the quantitative backbone of the t-SNE embedding) — https://github.com/matchms/ms2deepscore
- **MS2Query** (Supplies the pretrained model files (positive/negative mode) required by the importing pipeline to compute ms2deepscore scores)
- **specXplore** (Importing pipeline and interactive dashboard that wraps the t-SNE embedding of the ms2deepscore matrix with network views, heatmaps, and fragmentation overview maps) — https://github.com/kevinmildau/specXplore

## Examples

```
conda create --name specxplore_environment python=3.8 && conda activate specxplore_environment && pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- tsne_embedding.csv exists, is well-formed, and contains exactly one coordinate row (x, y) per wheat feature
- Selected t-SNE perplexity falls in the article's tested range (5–50) and the choice balances distance preservation against qualitative grouping
- Expert visual review confirms high-similarity features cluster locally around the Procyanidin-A2 corresponding feature, matching the paper's wheat example
- ms2deepscore scores were computed on a system not affected by ms2deepscore issue 199 (macos arm64), or cross-checked against another platform
- K-medoid clustering of the same matrix shows good agreement with the t-SNE projection, as reported in the article

## Limitations

- On macos arm64 systems, current ms2deepscore versions may produce similarity predictions inconsistent with other systems (ms2deepscore issue 199) without any error or warning, silently affecting the embedding
- ms2deepscore can only be expected to work well for spectra and metabolites well covered by or related to its training data; no coverage check is reported
- specXplore currently requires .mgf-formatted MS/MS data and works on macOS and Linux but fails on Windows

## Evidence

- [methods] Import MS/MS spectral data in .mgf format using matchms: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [methods] Generate t-SNE embedding of ms2deepscore similarity matrix: "based on a t-SNE embedding of the ms2deepscore pairwise similarity matrix serving as the primary similarity score."
- [methods] Perplexity tuning for t-SNE: "test a range of values for the perplexity tuning parameter (usually between 5 and 50) and select an appropriate value."
- [readme] feature identifier key requirement: [evidence span withheld — anchor status: external_doc]
- [results] t-SNE overview readability limitation: "On its own, the t-SNE overview figure is difficult to read. Only a limited view of clustering trends can be observed"
- [readme] macos arm64 ms2deepscore reliability warning: [evidence span withheld — anchor status: external_doc]
