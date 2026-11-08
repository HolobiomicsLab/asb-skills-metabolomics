---
name: pairwise-spectral-similarity-matrix-analysis
description: Use when exploring an MS/MS feature set via its pairwise spectral similarity
  matrix — e.g. you have a .mgf-derived specXplore session object and want threshold-tunable
  local network views, heatmaps, and clustering overlays rather than fixed molecular
  networking output.
when_to_use_negative:
- You need a fixed, standard molecular networking output (e.g. classical GNPS molecular
  networking) rather than adjustable, exploratory local views — the paper positions
  specXplore as a complement, not a replacement.
- Your data set exceeds roughly 5000 features — the authors only anticipate good scaling
  to 5000 features and expect difficulties for larger data sets and liberal settings.
- Your metabolites/spectra lie outside or are poorly related to ms2deepscore's training
  data — the pretrained ML score cannot be expected to work well there.
- You are running on Windows — the current specXplore version fails on Windows; workflows
  rely on macOS/Linux (and Anaconda PowerShell workarounds).
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: specXplore
  role: Interactive Python dashboard providing the t-SNE overview, k-medoid overlay,
    partial network drawings, similarity heatmaps, and fragmentation overview maps
    used to explore the pairwise similarity matrix
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports .mgf MS/MS spectral data into Python and performs basic specXplore
    data processing (including adding/renaming the "feature_id" metadata key) before
    similarity computation
  repo: https://matchms.readthedocs.io/en/latest/
- name: ms2deepscore
  role: Computes the pairwise spectral similarity matrix used as the primary similarity
    score and the basis for the t-SNE embedding
- name: MS2Query
  role: Provides pre-trained machine learning models used in the importing pipeline
    for the wheat and urine LC–MS/MS example data sets
- name: kmedoids
  role: Python package used for k-medoid clustering of spectral features over a range
    of k values to overlay on the t-SNE overview
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:9a5a9aa416f4b303247855d3803e3d627e309c129ee1958056746515946ba5dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - kmedoids
  - matchms
  - ms2deepscore
  - pandas
  - specXplore
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pairwise-spectral-similarity-matrix-analysis

## Summary

Analyze the raw pairwise spectral similarity matrix (ms2deepscore as primary score) directly within the specXplore dashboard, using the t-SNE embedding as an overview, k-medoid clustering overlays for grouping, and adjustable similarity thresholds plus top-K connectivity limits for localized partial network and heatmap exploration of LC-MS/MS features.

## When to use

Use when exploring an MS/MS feature set via its pairwise spectral similarity matrix — e.g. you have a .mgf-derived specXplore session object and want threshold-tunable local network views, heatmaps, and clustering overlays rather than fixed molecular networking output.

## When NOT to use

- You need a fixed, standard molecular networking output (e.g. classical GNPS molecular networking) rather than adjustable, exploratory local views — the paper positions specXplore as a complement, not a replacement.
- Your data set exceeds roughly 5000 features — the authors only anticipate good scaling to 5000 features and expect difficulties for larger data sets and liberal settings.
- Your metabolites/spectra lie outside or are poorly related to ms2deepscore's training data — the pretrained ML score cannot be expected to work well there.
- You are running on Windows — the current specXplore version fails on Windows; workflows rely on macOS/Linux (and Anaconda PowerShell workarounds).

## Inputs

- .mgf file with MS/MS spectral data, with feature identifier key "feature_id"
- specXplore session data object produced by the specXplore importing pipeline
- ms2deepscore pairwise similarity matrix (with feature identifiers)
- ms2query pre-trained model and library files for positive and negative mode (e.g. from Zenodo records 10527997 and 10528030)

## Outputs

- t-SNE embedding of the ms2deepscore pairwise similarity matrix (perplexity 5–50 range tested)
- k-medoid cluster assignments per feature for each tested k
- partial network drawings of the local neighborhood around a feature or feature group
- similarity heatmaps (matrix representations) of the raw pairwise similarity data
- fragmentation overview maps across spectra

## How to apply

First run the specXplore importing pipeline on the .mgf file (feature identifier key must be "feature_id"); the pipeline imports spectra with matchms, computes pairwise similarities with ms2deepscore (primary), modified cosine, and spec2vec, and saves a specXplore session data object. In the dashboard, start from the two-dimensional t-SNE embedding of the ms2deepscore pairwise similarity matrix, tuning the perplexity parameter over a range (usually between 5 and 50) to select an appropriate value. Overlay k-medoid clustering computed from the pairwise similarity matrix to complement the t-SNE overview; test a range of k values and pick groupings that agree with the t-SNE projection and topological insights (no automatic tuning guideline is provided). For local exploration, quickly adjust the similarity threshold and connectivity limitation settings — including pairwise similarity thresholds with corresponding minimum fragmentation overlaps (default 0.7), top-K neighbor limits, and maximal molecular family sizes — to regenerate partial network drawings and matrix (heatmap) representations around a feature or feature group of interest, visualizing edges only on demand under stringent user-modifiable filters.

## Related tools

- **specXplore** (Interactive Python dashboard providing the t-SNE overview, k-medoid overlay, partial network drawings, similarity heatmaps, and fragmentation overview maps used to explore the pairwise similarity matrix) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports .mgf MS/MS spectral data into Python and performs basic specXplore data processing (including adding/renaming the "feature_id" metadata key) before similarity computation) — https://matchms.readthedocs.io/en/latest/
- **ms2deepscore** (Computes the pairwise spectral similarity matrix used as the primary similarity score and the basis for the t-SNE embedding)
- **MS2Query** (Provides pre-trained machine learning models used in the importing pipeline for the wheat and urine LC–MS/MS example data sets)
- **kmedoids** (Python package used for k-medoid clustering of spectral features over a range of k values to overlay on the t-SNE overview)

## Examples

```
conda activate specxplore_environment; pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- The importing pipeline has produced a saved specXplore session data object that loads directly into a dashboard session (README: pipeline output 'can be fed directly into a specxplore dashboard session instance').
- Perplexity was swept over the recommended range (between 5 and 50) and the selected t-SNE embedding was judged appropriate before local exploration.
- k-medoid assignments are nondegenerate across the tested k range and agree qualitatively with the t-SNE projection and topological insights (paper reports good agreement).
- Edge display is gated by user-modifiable filters: pairwise similarity threshold with a corresponding minimum fragmentation overlap (default 0.7) and top-K neighbor limits; edges are only visualized on demand.
- Preprocessing stays tractable — e.g. <15 min for the wheat data set (<1000 features) and <75 min for the urine data set (<4000 features), including ~62 min MS2Query and 9 min pairwise similarity computation.

## Limitations

- ms2deepscore scores are only reliable for spectra/metabolites covered by or related to its training data; no coverage check is reported.
- macOS arm64 systems may produce unreliable ms2deepscore predictions due to matchms/ms2deepscore issue 199 — silent numerical divergence with no error or warning.
- Hierarchical clustering with medoid linkage was not used because of lacking Python implementations and suboptimality of hierarchically constrained clusters at any level of k.
- k-medoid assignments are not enforced to agree across different k values; cluster composition can change between k settings, and no optimality measure guides automatic cluster tuning.
- Dashboard scaling beyond ~5000 features is untested; larger data sets with liberal settings are expected to cause difficulties.

## Evidence

- [methods] Start from t-SNE embedding of ms2deepscore matrix: [evidence span withheld — anchor status: not_found]
- [methods] Perplexity range for tuning: "test a range of values for the perplexity tuning parameter (usually between 5 and 50) and select an appropriate value."
- [abstract] Threshold and connectivity settings are user-adjustable: "while allowing fast changes of threshold and connectivity limitation settings, providing flexibility in adjusting settings to suit the localized node environment being explored"
- [discussion] Default fragmentation-overlap edge filter: "the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps"
- [discussion] Edges only on demand: "avoided by only visualizing edges on demand, using stringent user-modifiable edge filter settings."
- [results] k-medoid works directly from the similarity matrix: "K-medoid clustering tends to show good agreement with both t-SNE projections and topological insights"
- [readme] Importing pipeline writes a session object for the dashboard: [evidence span withheld — anchor status: external_doc]
- [readme] Feature id key requirement: [evidence span withheld — anchor status: external_doc]
