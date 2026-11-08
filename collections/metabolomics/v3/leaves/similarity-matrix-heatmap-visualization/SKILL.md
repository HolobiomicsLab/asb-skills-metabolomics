---
name: similarity-matrix-heatmap-visualization
description: Use when a local, quantitative read-out of the raw pairwise similarity
  matrix is needed around selected features — e.g. to tune similarity thresholds or
  complement partial network views of LC-MS/MS feature data.
when_to_use_negative:
- The dataset contains more than ~5000 features — specXplore scaling beyond ~5000
  features is untested and performance issues are expected for larger data sets with
  liberal settings
- The input is raw vendor data or a feature table rather than a .mgf file with an
  MS/MS spectrum per feature; MZmine or vendor software must first export MGF
- The goal is classical global molecular networking on GNPS servers rather than local,
  threshold-tunable exploration of the raw pairwise similarity matrix
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: specXplore
  role: Python dashboard providing the interactive similarity heatmap view and threshold
    adjustment UI
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports .mgf spectral data into Python and performs basic specXplore data
    processing before scoring
  repo: https://matchms.readthedocs.io/en/latest/
- name: ms2deepscore
  role: Supplies the primary ms2deepscore pairwise similarities rendered as the heatmap's
    quantitative backbone
  repo: https://github.com/matchms/ms2deepscore
- name: spec2vec
  role: Provides the spec2vec score matrix whose implied adjacency is overlaid as
    markers
- name: MS2Query
  role: Supplies pretrained models used to compute the ms2deepscore similarities
- name: plotly
  role: Rendering engine for the interactive heatmap with hover panels and threshold-centered
    color scale
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:9336390375f8bda30757e06665928af067b8e1254beecc48716c1f57bff52d62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - plotly
  - spec2vec
  - specXplore
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# similarity-matrix-heatmap-visualization

## Summary

Render the pairwise spectral similarity matrix as an interactive plotly heatmap (augmap-style) in which ms2deepscore values form the quantitative backbone, with implied-adjacency markers for modified cosine and spec2vec scores overlaid at the current threshold.

## When to use

Use when a local, quantitative read-out of the raw pairwise similarity matrix is needed around selected features — e.g. to tune similarity thresholds or complement partial network views of LC-MS/MS feature data.

## When NOT to use

- The dataset contains more than ~5000 features — specXplore scaling beyond ~5000 features is untested and performance issues are expected for larger data sets with liberal settings
- The input is raw vendor data or a feature table rather than a .mgf file with an MS/MS spectrum per feature; MZmine or vendor software must first export MGF
- The goal is classical global molecular networking on GNPS servers rather than local, threshold-tunable exploration of the raw pairwise similarity matrix

## Inputs

- .mgf file with MS/MS feature spectra (feature identifier key 'feature_id')
- Precomputed pairwise similarity matrices for ms2deepscore, modified cosine, and spec2vec
- User-adjustable similarity threshold and connectivity limitation settings
- MS2Query pretrained model and library files (positive mode: zenodo record 10527997; negative mode: record 10528030)

## Outputs

- Interactive plotly heatmap (augmap) figure of the ms2deepscore similarity matrix with divergent color scale centered on the threshold
- Saved heatmap image, e.g. augmap_heatmap.png
- Hover panels exposing exact numeric ms2deepscore values per cell
- Overlay markers encoding modified cosine and spec2vec implied adjacency matrices at the current threshold

## How to apply

Import MS/MS spectra from the .mgf file into Python with matchms and run specXplore-style preprocessing, ensuring the feature identifier key is 'feature_id'. Compute the full pairwise similarity matrices for the three specXplore scores — ms2deepscore (via MS2Query pretrained models for positive or negative mode), modified cosine, and spec2vec. Render the matrix as a plotly heatmap using a divergent color scale centered on the current similarity threshold so that departures from the threshold are visually symmetric. Expose exact numeric ms2deepscore values through mouse-hover panels, and overlay distinguishable markers encoding the implied adjacency (above-threshold) relationships of the modified cosine and spec2vec scores at the same threshold. Rationale: in specXplore the similarity heatmap complements partial network drawings by exposing the quantitative backbone directly, letting users carefully expand neighborhood size via fast threshold changes on localized node selections.

## Related tools

- **specXplore** (Python dashboard providing the interactive similarity heatmap view and threshold adjustment UI) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports .mgf spectral data into Python and performs basic specXplore data processing before scoring) — https://matchms.readthedocs.io/en/latest/
- **ms2deepscore** (Supplies the primary ms2deepscore pairwise similarities rendered as the heatmap's quantitative backbone) — https://github.com/matchms/ms2deepscore
- **spec2vec** (Provides the spec2vec score matrix whose implied adjacency is overlaid as markers)
- **MS2Query** (Supplies pretrained models used to compute the ms2deepscore similarities)
- **plotly** (Rendering engine for the interactive heatmap with hover panels and threshold-centered color scale)

## Examples

```
conda activate specxplore_environment; jupyter-notebook notebooks/demo.ipynb
```

## Evaluation signals

- The heatmap figure exists and uses a divergent color scale centered on the current similarity threshold setting
- Hovering a cell reveals the exact numeric ms2deepscore value for that spectrum pair
- Implied-adjacency markers for modified cosine and spec2vec are both present and visually distinguishable from each other at the same threshold
- The rendered matrix dimensionality matches the number of features in the input .mgf file (specXplore worked well for <1000 and <4000 features in the paper)
- Threshold changes regenerate the local view (color center and markers) rather than requiring recomputation of the similarity matrices

## Limitations

- On macOS arm64 systems, ms2deepscore issue 199 may yield similarity predictions inconsistent with other systems without any error or warning, making results unreliable on affected systems
- No quantitative accuracy criterion is reported for the heatmap/augmap view in the paper; acceptance is by visual/interactive properties
- ms2deepscore can only be expected to work well for spectra and metabolites well covered by or related to its training data
- The current specXplore version works on macOS and Linux but fails on Windows

## Evidence

- [intro] specXplore provides means for (a) investigating the raw pairwise similarity matrix directly: "specXplore provides means for (a) investigating the raw pairwise similarity matrix directly, (b) investigating the fragmentation overlaps across multiple spectra"
- [methods] Compute pairwise spectral similarities with ms2deepscore, modified cosine, and spec2vec: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [abstract] ms2deepscore as quantitative backbone with fast threshold changes: "SpecXplore makes use of of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone, while allowing fast changes of threshold and connectivity limitation settings"
- [readme] similarity heatmaps are one of the dashboard's visualizations: [evidence span withheld — anchor status: external_doc]
- [readme] ms2deepscore arm64 reliability issue: [evidence span withheld — anchor status: external_doc]
