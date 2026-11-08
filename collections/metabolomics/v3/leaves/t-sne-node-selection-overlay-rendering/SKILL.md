---
name: t-sne-node-selection-overlay-rendering
description: Use when exploring LC–MS/MS spectra in a .mgf-based ms2deepscore t-SNE
  embedding and you need to see the local connectivity (adjacent nodes and edge weights)
  around one feature of interest before widening the neighborhood.
when_to_use_negative:
- When you need a full global molecular network of all spectra (specXplore deliberately
  visualizes edges only on demand with stringent user-modifiable filters, not complete
  network drawings).
- When the dataset exceeds roughly 5000 features — the paper anticipates the dashboard
  will scale to 5000 features but face difficulties for larger data sets under liberal
  settings.
- When input spectra are not in .mgf format or lack a 'feature_id' metadata key; the
  importing pipeline will not process them without conversion (e.g. via MZmine3 export
  or matchms.Spectrum key renaming).
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Interactive Python dashboard providing the t-SNE overview map, click-selection
    of a node, and on-demand ego-network edge overlay rendering with adjustable threshold
    and connectivity settings
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Supplies the pairwise spectral similarity scores that form the quantitative
    backbone of the t-SNE embedding and of the edges shown in the selected-node overlay
- name: dash
  role: Framework used to implement the visual analysis dashboard in which the overlay
    interactions (selection, threshold sliders, re-rendering) take place
- name: matchms
  role: Imports the .mgf MS/MS spectra into Python and attaches the required 'feature_id'
    metadata key during preprocessing, before the t-SNE map exists for selection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:05b4c98ecb9baf74c1bc6ec4047663d4b86b90ac6cfeabae5a6aa76789549aba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kevinmildau/specXplore
  tools:
  - Python
  - dash
  - matchms
  - ms2deepscore
  - specXplore
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# t-sne-node-selection-overlay-rendering

## Summary

An interactive specXplore technique in which a single node selected on the t-SNE overview map of ms2deepscore pairwise similarities is overlaid with ego-network edges revealing its adjacent neighbors, whose count is governed by user-adjustable similarity threshold and connectivity limitation settings.

## When to use

Use when exploring LC–MS/MS spectra in a .mgf-based ms2deepscore t-SNE embedding and you need to see the local connectivity (adjacent nodes and edge weights) around one feature of interest before widening the neighborhood.

## When NOT to use

- When you need a full global molecular network of all spectra (specXplore deliberately visualizes edges only on demand with stringent user-modifiable filters, not complete network drawings).
- When the dataset exceeds roughly 5000 features — the paper anticipates the dashboard will scale to 5000 features but face difficulties for larger data sets under liberal settings.
- When input spectra are not in .mgf format or lack a 'feature_id' metadata key; the importing pipeline will not process them without conversion (e.g. via MZmine3 export or matchms.Spectrum key renaming).

## Inputs

- specXplore session data object (produced by the importing pipeline from an .mgf MS/MS spectral file with a 'feature_id' key)
- precomputed ms2deepscore pairwise spectral similarity matrix (row/column index mapping feature identifiers)
- t-SNE embedding of the ms2deepscore similarity matrix displayed as the overview network
- user-adjustable edge filter parameters: pairwise similarity threshold with minimum fragmentation overlap (default 0.7), top-K neighbor limits, maximal molecular family sizes

## Outputs

- interactive t-SNE overview with an edge overlay showing the selected node's adjacent neighbors and connection weights
- regenerated local (ego) network view reflecting the chosen similarity threshold and connectivity limitation settings

## How to apply

Open the preprocessed specXplore dashboard session whose t-SNE overview is derived from the ms2deepscore pairwise similarity matrix of the imported .mgf spectra. Click-select the single reference (ego) node of interest on the t-SNE map; specXplore then renders edges on demand rather than drawing all edges at once, to avoid dense, unreadable topology. Adjust the similarity threshold settings (e.g. the default 0.7 ms2deepscore pairwise threshold with corresponding minimum fragmentation overlaps) and connectivity limitations such as top-K neighbor limits on individual nodes and maximal molecular family sizes, then let the overlay regenerate so the local view reflects the new constraints. Rationale: edges are only visualized under stringent user-modifiable filter settings, allowing careful expansion of the neighborhood size around the node of interest by progressively relaxing thresholds. If the neighborhood appears too sparse or too dense, change the threshold or top-K parameters and re-render rather than trusting the first view.

## Related tools

- **specXplore** (Interactive Python dashboard providing the t-SNE overview map, click-selection of a node, and on-demand ego-network edge overlay rendering with adjustable threshold and connectivity settings) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Supplies the pairwise spectral similarity scores that form the quantitative backbone of the t-SNE embedding and of the edges shown in the selected-node overlay)
- **dash** (Framework used to implement the visual analysis dashboard in which the overlay interactions (selection, threshold sliders, re-rendering) take place)
- **matchms** (Imports the .mgf MS/MS spectra into Python and attaches the required 'feature_id' metadata key during preprocessing, before the t-SNE map exists for selection)

## Examples

```
conda activate specxplore_environment && jupyter-notebook notebooks/demo.ipynb  # open demo, point data mgf to your .mgf file, run importing pipeline, then launch the specXplore dashboard to click-select a node on the t-SNE map
```

## Evaluation signals

- The overlay must show edges only for the selected (ego) node and its immediate neighbors — the article describes ego-network views as edge overlays that 'reveal which nodes are adjacent to a selected node', not full-network drawings.
- Changing the similarity threshold or top-K setting should visibly regenerate the local view; if the rendered neighborhood is identical after a parameter change, the edge filter was not actually applied.
- The set of displayed edges must exactly match a direct re-extraction from the thresholded ms2deepscore matrix at the stated threshold, top-K, and family-size constraints (structural consistency check, since no independent benchmark for this view is reported).
- The t-SNE background must correspond to the ms2deepscore similarity matrix embedding (not modified-cosine or spec2vec scores, which are computed but do not drive the overview map).
- Edge counts in the overlay should remain tractable: with liberal thresholds the view can become a dense topology, so a sane render shows a sparse neighborhood consistent with the default 0.7 threshold and top-K limits.

## Limitations

- ms2deepscore similarities — and hence the overlay edges — can only be expected to work well for spectra and metabolites well covered by or related to its training data; no coverage check is reported.
- On macos arm64 systems, a known ms2deepscore issue (matchms/ms2deepscore#199) can silently produce similarity predictions that are not in accordance with other systems, without errors or warnings, making the ms2deepscore results — and thus the overlay — unreliable.
- Performance for large data sets is untested; the dashboard is expected to scale to ~5000 features but face difficulties for larger data sets and liberal settings.
- specXplore works on macOS and Linux but fails on Windows; installation requires conda, python 3.8, and C++ compilers for the Cython backend.

## Evidence

- [results] t-SNE node selection overlay: "provides a two dimensional t-SNE embedding as a jumping board for local connectivity exploration using complementary interactive visualizations in the form of partial network drawings"
- [discussion] on-demand edge rendering: "avoided by only visualizing edges on demand, using stringent user-modifiable edge filter settings."
- [abstract] threshold and connectivity settings: "while allowing fast changes of threshold and connectivity limitation settings, providing flexibility in adjusting settings to suit the localized node environment being explored"
- [discussion] default similarity threshold: "the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps"
- [intro] neighborhood expansion rationale: "the ability to quickly change thresholds to regenerate local views under the new constraints, allowing the careful expansion of neighborhood size"
- [readme] backbone similarity score: [evidence span withheld — anchor status: external_doc]
- [readme] ms2deepscore reliability caveat: [evidence span withheld — anchor status: external_doc]
