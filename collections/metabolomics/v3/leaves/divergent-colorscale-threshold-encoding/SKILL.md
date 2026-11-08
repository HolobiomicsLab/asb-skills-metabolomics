---
name: divergent-colorscale-threshold-encoding
description: Use when quantitative pairwise LC–MS/MS similarity matrices (ms2deepscore,
  modified cosine, spec2vec) must be inspected cell-by-cell around a tunable similarity
  threshold, especially for small node selections where network views are too coarse.
when_to_use_negative:
- The dataset exceeds roughly 5000 features — the paper only anticipates that the
  dashboard scales well to 5000 features and expects difficulties beyond that, so
  full-matrix heatmaps become impractical.
- The analysis goal is global network topology or cluster structure; the article presents
  heatmaps as complements to (not replacements for) partial network drawings and the
  t-SNE overview.
- Input is raw vendor LC–MS/MS data rather than a processed .mgf file with feature_id
  metadata — specXplore requires .mgf output (e.g. from MZmine3 export) first.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3172
tools:
- name: matchms
  role: imports the .mgf MS/MS spectra into Python and supports adding/renaming the
    'feature_id' metadata key before similarity computation
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: provides the primary pairwise similarity values that are encoded quantitatively
    in the heatmap; pretrained models come from MS2Query
  repo: https://github.com/kevinmildau/specXplore
- name: MS2Query
  role: supplies the pretrained model files (positive and negative mode, via Zenodo)
    needed to compute ms2deepscore similarities
- name: spec2vec
  role: one of the two comparison scores whose implied adjacency at the threshold
    is overlaid as markers on the heatmap
  repo: https://github.com/kevinmildau/specXplore
- name: plotly
  role: renders the interactive augmap heatmap with divergent color scale and hover
    panels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:5e727ed017ea0a4a8509f31f67537be79f3e5d8981c4c4622381a33e494a5bf9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# divergent-colorscale-threshold-encoding

## Summary

Render an augmap-style pairwise similarity heatmap (specXplore's quantitative backbone visualization) with a divergent color scale centered on the current similarity threshold, exposing exact numeric ms2deepscore values on hover and overlaying markers that encode the implied adjacency of modified cosine and spec2vec scores at that threshold.

## When to use

Use when quantitative pairwise LC–MS/MS similarity matrices (ms2deepscore, modified cosine, spec2vec) must be inspected cell-by-cell around a tunable similarity threshold, especially for small node selections where network views are too coarse.

## When NOT to use

- The dataset exceeds roughly 5000 features — the paper only anticipates that the dashboard scales well to 5000 features and expects difficulties beyond that, so full-matrix heatmaps become impractical.
- The analysis goal is global network topology or cluster structure; the article presents heatmaps as complements to (not replacements for) partial network drawings and the t-SNE overview.
- Input is raw vendor LC–MS/MS data rather than a processed .mgf file with feature_id metadata — specXplore requires .mgf output (e.g. from MZmine3 export) first.

## Inputs

- .mgf file with MS/MS feature spectra (feature identifier key 'feature_id')
- ms2deepscore pairwise similarity matrix computed from the imported spectra
- modified cosine pairwise similarity matrix
- spec2vec pairwise similarity matrix
- current similarity threshold setting

## Outputs

- augmap heatmap figure (plotly, divergent color scale centered on the threshold)
- exported static image, e.g. augmap_heatmap.png

## How to apply

Import MS/MS spectra from the .mgf file with matchms and run basic specXplore-style preprocessing, computing the full pairwise similarity matrices with ms2deepscore (pretrained MS2Query models), modified cosine, and spec2vec. Render the augmap with plotly, choosing a divergent color scale whose midpoint sits exactly at the current similarity threshold so cells above/below the cutoff are visually separated. Attach hover panels that expose exact numeric ms2deepscore values rather than color-only encoding, since the heatmap serves as the quantitative complement to t-SNE and network views. Overlay distinguishable markers encoding the implied adjacency matrices of the modified cosine and spec2vec scores under the same threshold, enabling side-by-side score comparison. Because threshold and connectivity-limitation settings change rapidly during exploration, keep the figure regenerable so local views can be redrawn under new constraints.

## Related tools

- **matchms** (imports the .mgf MS/MS spectra into Python and supports adding/renaming the 'feature_id' metadata key before similarity computation) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (provides the primary pairwise similarity values that are encoded quantitatively in the heatmap; pretrained models come from MS2Query) — https://github.com/kevinmildau/specXplore
- **MS2Query** (supplies the pretrained model files (positive and negative mode, via Zenodo) needed to compute ms2deepscore similarities)
- **spec2vec** (one of the two comparison scores whose implied adjacency at the threshold is overlaid as markers on the heatmap) — https://github.com/kevinmildau/specXplore
- **plotly** (renders the interactive augmap heatmap with divergent color scale and hover panels)

## Examples

```
import plotly.graph_objects as go; fig = go.Figure(go.Heatmap(z=scores_ms2deepscore, zmid=threshold, colorscale='RdBu')); fig.write_image('augmap_heatmap.png')
```

## Evaluation signals

- The heatmap figure exists and is saved as augmap_heatmap.png.
- The color scale is divergent and centered on the current similarity threshold, so score values at the cutoff map to the scale midpoint.
- Mouse-hover panels expose exact numeric ms2deepscore values, not just color.
- Implied-adjacency markers for modified cosine and spec2vec at the same threshold are present and distinguishable from each other.
- The rendered matrix dimensions match the number of imported features in the .mgf file.

## Limitations

- No quantitative accuracy criterion is reported for this view in the paper; validation is limited to structural acceptance checks (divergent scale, hover values, overlay markers).
- On macOS arm64 machines, ms2deepscore may produce similarity predictions inconsistent with other systems (matchms/ms2deepscore issue 199) without any error or warning, making the quantitative readout unreliable.
- ms2deepscore is only expected to work well for spectra/metabolites covered by or related to its training data; no coverage check is reported, and specXplore is expected to face difficulties for datasets larger than ~5000 features.

## Evidence

- [methods] The pipeline computes three similarity scores for the heatmap: ms2deepscore as primary plus modified cosine and spec2vec overlays.: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [abstract] Threshold adjustment is a core interactive capability that the divergent-scale encoding is designed around.: "while allowing fast changes of threshold and connectivity limitation settings, providing flexibility in adjusting settings to suit the localized node environment being explored"
- [readme] Similarity heatmaps are one of the complementary views in specXplore that this skill renders.: [evidence span withheld — anchor status: external_doc]
- [methods] Spectra must be imported with matchms and preprocessed before the pairwise matrices driving the heatmap exist.: "Spectral data are imported into Python using matchms, and basic specXplore data processing is performed"
- [methods] The heatmap is a complement to the ms2deepscore-based t-SNE overview, not a standalone entry point.: "based on a t-SNE embedding of the ms2deepscore pairwise similarity matrix serving as the primary similarity score."
