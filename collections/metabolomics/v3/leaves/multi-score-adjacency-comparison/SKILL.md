---
name: multi-score-adjacency-comparison
description: Use when mgf LC–MS/MS feature spectra need a quantitative heatmap readout
  of ms2deepscore similarities with modified cosine and spec2vec adjacency overlays
  to guide local threshold adjustment or replace network views for small node selections.
when_to_use_negative:
- Dataset contains more than ~5000 features — scalability beyond ~5000 features is
  untested and dashboard performance is expected to degrade
- Working on Windows — the specXplore pipeline 'currently works on Macos and Linux
  but fails in windows'
- Only a coarse global clustering overview is needed — the t-SNE overview or network
  views, not the quantitative augmap, are the intended entry point
edam_operation: http://edamontology.org/operation_0337
tools:
- name: specXplore
  role: Dashboard framework providing the importing pipeline and augmap/similarity
    heatmap views
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports .mgf spectral data into Python and performs basic specXplore preprocessing;
    also used to add/rename the 'feature_id' metadata key
  repo: https://matchms.readthedocs.io/en/latest/
- name: ms2deepscore
  role: Computes the ms2deepscore pairwise similarity matrix used as the quantitative
    backbone of the heatmap color scale
- name: MS2Query
  role: Supplies the pretrained machine-learning models used for ms2deepscore similarity
    predictions
- name: spec2vec
  role: Computes spec2vec pairwise scores whose implied adjacency is overlaid as markers
    on the augmap
- name: plotly
  role: Renders the interactive augmap heatmap with hover panels and adjacency overlays
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:a7a4c455e354958421fb8f91a480ad2e48408c73345fa66ae2ac7e280760573e
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# multi-score-adjacency-comparison

## Summary

Reconstructs specXplore-style augmap (similarity heatmap) views that portray ms2deepscore pairwise similarities quantitatively while overlaying the implied adjacency of modified cosine and spec2vec scores at the current threshold, complementing network views for local spectral similarity exploration.

## When to use

Use when .mgf LC–MS/MS feature spectra need a quantitative heatmap readout of ms2deepscore similarities with modified cosine and spec2vec adjacency overlays to guide local threshold adjustment or replace network views for small node selections.

## When NOT to use

- Dataset contains more than ~5000 features — scalability beyond ~5000 features is untested and dashboard performance is expected to degrade
- Working on Windows — the specXplore pipeline 'currently works on Macos and Linux but fails in windows'
- Only a coarse global clustering overview is needed — the t-SNE overview or network views, not the quantitative augmap, are the intended entry point

## Inputs

- MS/MS feature spectra in .mgf format with a 'feature_id' metadata key per spectrum
- MS2Query pretrained model files (positive mode: zenodo.org/records/10527997; negative mode: zenodo.org/records/10528030), placed in separate 'pos'/'neg' folders

## Outputs

- Rendered augmap heatmap figure (e.g. augmap_heatmap.png) with divergent color scale centered on the similarity threshold
- Full pairwise similarity matrices for ms2deepscore, modified cosine, and spec2vec scores
- Implied adjacency markers for modified cosine and spec2vec at the chosen threshold

## How to apply

Import MS/MS feature spectra from the .mgf file into Python using matchms and apply basic specXplore preprocessing, ensuring spectra carry a 'feature_id' metadata key. Compute the full pairwise similarity matrices with the three specXplore scores: ms2deepscore (using the pretrained MS2Query models, loaded per ionization mode from separate model folders), modified cosine, and spec2vec. Render the augmap with plotly using a divergent color scale centered on the current similarity threshold so deviations from the threshold are visually symmetric. Expose exact numeric ms2deepscore values via mouse-hover panels, and overlay distinguishable markers encoding the implied adjacency matrices of the modified cosine and spec2vec scores binarized at that threshold. Use the resulting view for local threshold adjustment or as an alternative to partial network drawings when only a small node selection is under inspection.

## Related tools

- **specXplore** (Dashboard framework providing the importing pipeline and augmap/similarity heatmap views) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports .mgf spectral data into Python and performs basic specXplore preprocessing; also used to add/rename the 'feature_id' metadata key) — https://matchms.readthedocs.io/en/latest/
- **ms2deepscore** (Computes the ms2deepscore pairwise similarity matrix used as the quantitative backbone of the heatmap color scale)
- **MS2Query** (Supplies the pretrained machine-learning models used for ms2deepscore similarity predictions)
- **spec2vec** (Computes spec2vec pairwise scores whose implied adjacency is overlaid as markers on the augmap)
- **plotly** (Renders the interactive augmap heatmap with hover panels and adjacency overlays)

## Examples

```
conda create --name specxplore_environment python=3.8; pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- The heatmap exists (e.g. augmap_heatmap.png) and uses a divergent color scale centered on the current similarity threshold
- Mouse-hover panels expose exact numeric ms2deepscore values for each feature pair
- Implied-adjacency markers for both modified cosine and spec2vec scores are present and visually distinguishable at the chosen threshold
- Each pairwise similarity matrix is square over the same feature set as the input .mgf spectra (matching 'feature_id' identifiers)
- The three score types match specXplore's default score set: ms2deepscore, modified cosine, and spec2vec

## Limitations

- No quantitative accuracy criterion is reported for the augmap view; acceptance is structural (divergent scale, hover values, adjacency markers)
- ms2deepscore applicability is limited to spectra/metabolites covered by or related to its training data; no coverage check is reported
- On macOS arm64 systems, ms2deepscore issue 199 can produce similarity predictions inconsistent with other systems, silently affecting the importing pipeline's default ms2deepscore scores
- Threshold-dependent adjacency markers are implied (binarized) rather than raw score readouts, so modified cosine and spec2vec values themselves are not shown numerically

## Evidence

- [methods] three similarity scores in methods: "automatically computes pairwise spectral similarities using three similarity scores: ms2deepscore, modified cosine scores, and spec2vec scores."
- [methods] pretrained MS2Query models: "For the machine learning scores, pretrained models provided with MS2Query are used."
- [readme] heatmap views in specXplore: [evidence span withheld — anchor status: external_doc]
- [readme] arm64 ms2deepscore reliability warning: [evidence span withheld — anchor status: external_doc]
- [discussion] ms2deepscore training-data coverage caveat: "it can only be expected to work well for spectra and metabolites well covered by or related to its training data"
