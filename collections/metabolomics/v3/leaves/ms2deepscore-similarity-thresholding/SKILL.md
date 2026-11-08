---
name: ms2deepscore-similarity-thresholding
description: Use when exploring LC-MS/MS feature connectivity around a node of interest
  in specXplore and you need to regenerate the local edge overlay under new threshold
  or connectivity-limitation constraints.
when_to_use_negative:
- When the input spectra/metabolites are not well covered by or related to the ms2deepscore
  training data — the score can only be expected to work well for such compounds
- When working with very large feature sets (>5000 features) or liberal settings —
  specXplore is expected to face difficulties scaling beyond ~5000 features
- 'When you need a single global, static molecular network: specXplore deliberately
  avoids visualizing edges by default and is built for local, on-demand edge visualization
  instead'
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: specXplore
  role: Interactive Python (Dash) dashboard in which the ms2deepscore similarity matrix
    is thresholded and visualized as on-demand edge overlays over the t-SNE embedding;
    thresholds and connectivity limitations are changed interactively to regenerate
    local views
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Provides the state-of-the-art pairwise spectral similarity scores that serve
    as the quantitative backbone being thresholded into edges
- name: matchms
  role: Imports the .mgf spectral data into Python and performs basic specXplore data
    processing prior to similarity computation
- name: dash
  role: Framework underlying the visual analysis dashboard through which threshold/edge-filter
    settings are exposed to the user
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:c26114393333b4ac828b95863982396e7b002351da322e73ba546b5752065bf9
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
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# ms2deepscore-similarity-thresholding

## Summary

Binarize the ms2deepscore pairwise spectral similarity matrix into edges using adjustable similarity thresholds (e.g. a 0.7 default with minimum fragmentation overlap), top-K neighbor limits, and maximal molecular family sizes, to render localized ego-networks in the specXplore t-SNE dashboard without overwhelming the view with dense topology.

## When to use

Use when exploring LC-MS/MS feature connectivity around a node of interest in specXplore and you need to regenerate the local edge overlay under new threshold or connectivity-limitation constraints.

## When NOT to use

- When the input spectra/metabolites are not well covered by or related to the ms2deepscore training data — the score can only be expected to work well for such compounds
- When working with very large feature sets (>5000 features) or liberal settings — specXplore is expected to face difficulties scaling beyond ~5000 features
- When you need a single global, static molecular network: specXplore deliberately avoids visualizing edges by default and is built for local, on-demand edge visualization instead

## Inputs

- ms2deepscore pairwise similarity matrix computed by the specXplore importing pipeline from an .mgf-format MS/MS feature list (with 'feature_id' keys)
- User-adjustable similarity threshold value with corresponding minimum fragmentation overlap (e.g. 0.7 default)
- Top-K neighbor limit per node and maximal molecular family size settings
- A selected ego/reference feature on the t-SNE embedding

## Outputs

- On-demand edge overlays on the t-SNE overview showing which nodes are adjacent to the selected node at the current threshold
- Regenerated partial network / local connectivity view reflecting the new threshold and connectivity-limitation settings
- Similarity heatmap or matrix representation of the thresholded local neighborhood

## How to apply

Start from the ms2deepscore pairwise similarity matrix computed during the specXplore importing pipeline (alongside modified cosine and spec2vec scores, with ms2deepscore as the primary similarity). Apply a pairwise similarity threshold together with a corresponding minimum fragmentation overlap (the paper cites a default of 0.7), then impose top-K neighbor limits on individual nodes and maximal molecular family sizes to prevent dense, unreadable topology. Adjust these settings on demand in the dashboard and regenerate the local view under the new constraints, allowing careful expansion of the neighborhood size around a feature of interest; edges are only visualized on demand using these stringent user-modifiable edge filter settings. Because edges are drawn as overlays on the t-SNE embedding (which itself is computed from the ms2deepscore matrix, with perplexity tuned typically between 5 and 50), a change in threshold should visibly change which neighbor nodes are connected to the selected ego node.

## Related tools

- **specXplore** (Interactive Python (Dash) dashboard in which the ms2deepscore similarity matrix is thresholded and visualized as on-demand edge overlays over the t-SNE embedding; thresholds and connectivity limitations are changed interactively to regenerate local views) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Provides the state-of-the-art pairwise spectral similarity scores that serve as the quantitative backbone being thresholded into edges)
- **matchms** (Imports the .mgf spectral data into Python and performs basic specXplore data processing prior to similarity computation)
- **dash** (Framework underlying the visual analysis dashboard through which threshold/edge-filter settings are exposed to the user)

## Examples

```
pip install "git+https://github.com/kevinmildau/specXplore.git"  # then open demo.ipynb; in the dashboard adjust similarity threshold (e.g. 0.7), min fragmentation overlap, top-K and family-size to regenerate the ego-network overlay
```

## Evaluation signals

- The rendered ego-network overlay should contain exactly the node pairs whose ms2deepscore similarity (with required minimum fragmentation overlap) meets the currently set threshold, and no others — changing the threshold must visibly add or remove edges around the selected node
- Per-node degree in the displayed sub-network must not exceed the configured top-K neighbor limit, and connected components must not exceed the maximal molecular family size setting
- Edge density should remain readable: the paper notes dense topology is avoided 'by only visualizing edges on demand, using stringent user-modifiable edge filter settings'
- The local view should be reproducible: re-entering the same threshold, fragmentation-overlap, top-K, and family-size settings must regenerate the identical overlay
- The t-SNE positions of nodes should remain fixed across threshold changes — only the overlay edges change, since the embedding is a one-time t-SNE of the full ms2deepscore matrix

## Limitations

- ms2deepscore can only be expected to work well for spectra and metabolites well covered by or related to its training data; no coverage check is performed by the pipeline
- On macOS arm64 systems, a known ms2deepscore issue (matchms/ms2deepscore#199) can produce similarity predictions inconsistent with other systems, silently affecting thresholded results — the README flags this explicitly
- The dashboard is expected to scale well to 5000 features but to face difficulties for larger data sets and liberal (low-threshold) settings
- No tool version numbers are stated for the underlying software, complicating exact reproduction of similarity scores
- specXplore currently works on macOS and Linux but fails on Windows without extra setup (Anaconda + MS Visual C++ Redistributable)

## Evidence

- [abstract] SpecXplore makes use of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone, while allowing fast changes of threshold and connectivity limitation settings: [evidence span withheld — anchor status: not_found]
- [intro] the ability to quickly change thresholds to regenerate local views under the new constraints, allowing the careful expansion of neighborhood size: "the ability to quickly change thresholds to regenerate local views under the new constraints, allowing the careful expansion of neighborhood size"
- [discussion] avoided by only visualizing edges on demand, using stringent user-modifiable edge filter settings: "avoided by only visualizing edges on demand, using stringent user-modifiable edge filter settings"
- [discussion] it can only be expected to work well for spectra and metabolites well covered by or related to its training data: "it can only be expected to work well for spectra and metabolites well covered by or related to its training data"
- [readme] The current ms2deepscore package version may lead to ms2deepscore similarity predictions that are not in accordance with results on other systems: [evidence span withheld — anchor status: external_doc]
