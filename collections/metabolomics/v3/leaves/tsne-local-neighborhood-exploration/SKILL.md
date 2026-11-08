---
name: tsne-local-neighborhood-exploration
description: Use when a precomputed pairwise MS/MS similarity matrix (e.g., ms2deepscore)
  and a .mgf feature set need interactive local exploration around a densely connected
  node to support structural hypothesis generation.
when_to_use_negative:
- Input is raw vendor data or an MGF has not yet been exported/annotated with a 'feature_id'
  key — run the specXplore importing pipeline or MZmine export first
- Dataset exceeds ~5000 features after processing; the dashboard is expected to face
  difficulties for larger datasets with liberal settings
- Target metabolites are poorly covered by or unrelated to the ms2deepscore training
  data; the scores cannot be expected to work well
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Interactive Python dashboard providing the t-SNE overview, threshold adjustment,
    partial network drawings, similarity heatmaps, and fragmentation overview maps
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Provides the state-of-the-art pairwise spectral similarities used as the quantitative
    backbone for the t-SNE embedding and neighborhood thresholding
  repo: https://github.com/kevinmildau/specXplore
- name: matchms
  role: Imports the .mgf MS/MS spectra into Python and performs basic specXplore data
    processing including feature_id metadata handling
  repo: https://github.com/kevinmildau/specXplore
- name: MS2Query
  role: Pretrained models supply machine-learning scores joined into the dashboard
    during the importing pipeline
  repo: https://github.com/kevinmildau/specXplore
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:0be6d474060a0567c0fe419c5cc557881f2a26852b39923e892c2780a07113ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - MS2Query
  - Python
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.88
n_claims: 25
---

# tsne-local-neighborhood-exploration

## Summary

Explores LC–MS/MS spectral similarity data by using a 2D t-SNE embedding of the ms2deepscore pairwise similarity matrix as an overview 'jumping board', then zooming into a local network neighborhood around a feature of interest (e.g., the Procyanidin-A2 dense region) with adjustable similarity thresholds and connectivity limits.

## When to use

Use when a precomputed pairwise MS/MS similarity matrix (e.g., ms2deepscore) and a .mgf feature set need interactive local exploration around a densely connected node to support structural hypothesis generation.

## When NOT to use

- Input is raw vendor data or an MGF has not yet been exported/annotated with a 'feature_id' key — run the specXplore importing pipeline or MZmine export first
- Dataset exceeds ~5000 features after processing; the dashboard is expected to face difficulties for larger datasets with liberal settings
- Target metabolites are poorly covered by or unrelated to the ms2deepscore training data; the scores cannot be expected to work well

## Inputs

- LC–MS/MS MS/MS spectral data in .mgf format with a 'feature_id' metadata key (e.g., urine metabolome polyphenol exposome study, <4000 features after processing)
- Precomputed ms2deepscore pairwise similarity matrix and t-SNE embedding from the specXplore importing pipeline
- MS2Query pretrained model and library files (positive or negative mode) in a models subdirectory

## Outputs

- Interactive t-SNE overview network of the full pairwise similarity matrix
- Partial network drawing, similarity heatmap, and fragmentation overview map for the selected local neighborhood
- k-medoid cluster assignments and metadata/chemical-classification overlays for the explored region

## How to apply

Run the specXplore importing pipeline (matchms import of .mgf spectra with a 'feature_id' key; ms2deepscore, modified cosine, and spec2vec similarities; MS2Query pretrained models) to build a session data object. Open the dashboard, locate the feature of interest on the t-SNE overview (tune perplexity, usually between 5 and 50), and threshold the ms2deepscore matrix (default minimum fragmentation overlap 0.7) to extract the local neighborhood. Adjust top-K neighbor limits and maximal molecular family sizes to expand or contract the neighborhood, and inspect partial network drawings, similarity heatmaps, and fragmentation overview maps alongside joined metadata. Rationale: the t-SNE overview alone is hard to read; local connectivity, fragmentation-pattern overlaps, and metadata together serve as starting points for manual annotation and structural hypothesis generation.

## Related tools

- **specXplore** (Interactive Python dashboard providing the t-SNE overview, threshold adjustment, partial network drawings, similarity heatmaps, and fragmentation overview maps) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Provides the state-of-the-art pairwise spectral similarities used as the quantitative backbone for the t-SNE embedding and neighborhood thresholding) — https://github.com/kevinmildau/specXplore
- **matchms** (Imports the .mgf MS/MS spectra into Python and performs basic specXplore data processing including feature_id metadata handling) — https://github.com/kevinmildau/specXplore
- **MS2Query** (Pretrained models supply machine-learning scores joined into the dashboard during the importing pipeline) — https://github.com/kevinmildau/specXplore

## Examples

```
conda create --name specxplore_environment python=3.8 && conda activate specxplore_environment && pip install "git+https://github.com/kevinmildau/specXplore.git"
```

## Evaluation signals

- The t-SNE overview must be built from the ms2deepscore pairwise similarity matrix as the primary similarity score
- Local edges require the pairwise threshold with a minimum fragmentation overlap (default 0.7); tightening/loosening thresholds must regenerate the local view
- K-medoid clustering should show good agreement with t-SNE projections and topological insights
- The urine dataset example works on <4000 features after processing (wheat <1000); preprocessing completed in <75 min for urine
- Cluster assignments are only valid for the chosen k; agreement across different k values is not enforced

## Limitations

- The t-SNE overview figure on its own is difficult to read; only limited clustering trends are visible without interactivity
- ms2deepscore works only for spectra/metabolites well covered by or related to its training data; no coverage check is reported
- The current version works on macOS and Linux but fails on Windows; on macOS arm64, ms2deepscore issue 199 can silently produce unreliable similarity predictions
- No online hosted version exists; the dashboard is local-only, and scaling beyond ~5000 features is untested

## Evidence

- [abstract] t-SNE jumping board: "provides a two dimensional t-SNE embedding as a jumping board for local connectivity exploration using complementary interactive visualizations"
- [abstract] adjustable settings: "while allowing fast changes of threshold and connectivity limitation settings, providing flexibility in adjusting settings to suit the localized node environment"
- [methods] perplexity tuning: "test a range of values for the perplexity tuning parameter (usually between 5 and 50) and select an appropriate value"
- [discussion] threshold/overlap coupling: "the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps"
- [discussion] training-data coverage: "it can only be expected to work well for spectra and metabolites well covered by or related to its training data"
- [discussion] scalability: "We expect that the dashboard will scale well to 5000 features but face difficulties for larger data sets and liberal settings."
- [readme] tool purpose: [evidence span withheld — anchor status: external_doc]
- [readme] mgf requirement: [evidence span withheld — anchor status: external_doc]
