---
name: specxplore-importing-pipeline-execution
description: Use when raw LC–MS/MS .mgf feature lists (e.g. wheat or urine metabolome
  datasets) must be converted into a specXplore session data object with pairwise
  similarity scores and a t-SNE embedding before interactive dashboard exploration.
when_to_use_negative:
- Input is already a processed specXplore session data object — it can be fed directly
  into the dashboard without re-running the importing pipeline.
- Data is not in .mgf format with a 'feature_id' metadata key — convert first (e.g.
  via MZmine3 or matchms.Spectrum key renaming) before applying this skill.
- Working on Windows without the Microsoft Visual C++ Redistributable installed —
  the Cython backend computation will fail.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Provides the importing pipeline that preprocesses .mgf data into a session
    data object
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports spectral data into Python and performs basic specXplore data processing
- name: MS2Query
  role: Executes pretrained-model library search and supplies models for machine-learning
    similarity scores
- name: ms2deepscore
  role: Computes pairwise ms2deepscore similarity scores used as the quantitative
    backbone
- name: spec2vec
  role: Computes pairwise spec2vec similarity scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:f0c8386cd354051103ca0885a48c75c71630b60d8340f0c7bc1f14e0f4dd4265
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - spec2vec
  - specXplore
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# specxplore-importing-pipeline-execution

## Summary

Run the specXplore importing pipeline to preprocess LC–MS/MS .mgf feature data into a specXplore session data object, computing ms2deepscore, modified cosine, and spec2vec pairwise similarity scores plus a t-SNE embedding via MS2Query-backed pretrained models before dashboard exploration.

## When to use

Use when raw LC–MS/MS .mgf feature lists (e.g. wheat or urine metabolome datasets) must be converted into a specXplore session data object with pairwise similarity scores and a t-SNE embedding before interactive dashboard exploration.

## When NOT to use

- Input is already a processed specXplore session data object — it can be fed directly into the dashboard without re-running the importing pipeline.
- Data is not in .mgf format with a 'feature_id' metadata key — convert first (e.g. via MZmine3 or matchms.Spectrum key renaming) before applying this skill.
- Working on Windows without the Microsoft Visual C++ Redistributable installed — the Cython backend computation will fail.

## Inputs

- .mgf formatted file with MS/MS spectral data (e.g. from MZmine3 GNPS-FBMN export)
- MS2Query pretrained model and library files (positive: Zenodo record 10527997; negative: Zenodo record 10528030)
- specXplore conda environment with Python 3.8 (conda activate specxplore_environment)

## Outputs

- specXplore session data object saved to the hard drive (consumable by the specXplore dashboard)
- Pairwise spectral similarity matrices for ms2deepscore, modified cosine, and spec2vec scores
- t-SNE embedding of the ms2deepscore pairwise similarity matrix

## How to apply

Install specXplore in a conda environment with Python 3.8 on macOS or Linux (Windows additionally requires the Anaconda power shell and Microsoft Visual C++ Redistributable for the Cython backend). Load the .mgf MS/MS feature file into Python with matchms, ensuring the feature identifier key is named 'feature_id' (rename via matchms.Spectrum or a text editor if needed). Run the demo.ipynb importing pipeline, which automatically computes three pairwise similarity scores — ms2deepscore, modified cosine, and spec2vec — and performs MS2Query pretrained-model library search (models from Zenodo, separated by ion mode into pos/neg folders). Generate a t-SNE embedding of the ms2deepscore similarity matrix, testing perplexity values usually between 5 and 50, then save the resulting session data object to the hard drive for direct feeding into a specXplore dashboard session. Expect the pipeline to take <15 min on ~1000-feature datasets like the wheat plants data and <75 min on ~4000-feature datasets like the urine metabolome, with ~62 min of that for MS2Query and ~9 min for pairwise spectral similarity computations on an Apple M1 Pro MacBook Pro (2021).

## Related tools

- **specXplore** (Provides the importing pipeline that preprocesses .mgf data into a session data object) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports spectral data into Python and performs basic specXplore data processing)
- **MS2Query** (Executes pretrained-model library search and supplies models for machine-learning similarity scores)
- **ms2deepscore** (Computes pairwise ms2deepscore similarity scores used as the quantitative backbone)
- **spec2vec** (Computes pairwise spec2vec similarity scores)

## Examples

```
conda create --name specxplore_environment python=3.8; conda activate specxplore_environment; pip install "git+https://github.com/kevinmildau/specXplore.git"; jupyter-notebook
```

## Evaluation signals

- Number of features after processing matches the dataset scale: <1000 for wheat plants LC–MS/MS data and <4000 for urine metabolome data.
- Pipeline wall-clock time is <15 min on wheat-scale data and <75 min on urine-scale data (Apple M1 Pro MacBook Pro, 2021).
- All three similarity scores (ms2deepscore, modified cosine, spec2vec) are computed and present in the session data object.
- t-SNE embedding is generated from the ms2deepscore similarity matrix with a selected perplexity in the 5–50 range.
- Session data object loads successfully into a specXplore dashboard session instance.

## Limitations

- Current specXplore version fails on Windows; only macOS and Linux are supported.
- On macOS arm64 (Apple Silicon), ms2deepscore issue 199 can produce similarity predictions inconsistent with other systems without any error or warning, making results unreliable.
- The .mgf file must contain the feature identifier key 'feature_id'; any other key must be renamed beforehand.
- MS2Query model and library files must be downloaded separately and organized into pos/neg subfolders.

## Evidence

- [methods] Before data can be opened in the specXplore dashboard, it needs to be processed using the specXplore importing pipeline.: "Before data can be opened in the specXplore dashboard, it needs to be processed using the specXplore importing pipeline."
- [methods] Spectral data are imported into Python using matchms: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [methods] automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores.: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [results] the processing time for the larger urine data set was less than 75 min: "the processing time for the larger urine data set was less than 75 min, whereof approximately 62 min was taken up by MS2Query and 9 min for all pairwise spectral similarity computations"
- [readme] the current version of specXplore works on Macos and Linux but fails in windows.: [evidence span withheld — anchor status: external_doc]
- [readme] users making use of macos arm64 computers should be aware of issue 199 for ms2deepscore: [evidence span withheld — anchor status: external_doc]
- [readme] specXplore currently requires a .mgf formatted file with MS/MS spectral data: [evidence span withheld — anchor status: external_doc]
