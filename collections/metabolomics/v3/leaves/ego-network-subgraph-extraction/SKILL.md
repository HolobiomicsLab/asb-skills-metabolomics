---
name: ego-network-subgraph-extraction
description: Use when a single LC-MS/MS feature (node) in the t-SNE overview needs
  its local neighborhood examined — i.e., the user wants to see which features are
  adjacent under a given similarity threshold, and optionally expand that neighborhood
  by regenerating under new constraints.
when_to_use_negative:
- When the goal is a global molecular-network overview across the whole feature set
  — specXplore's t-SNE plus edge overlays are explicitly designed for localized, ego-centric
  exploration, not exhaustive network rendering (the paper warns that liberal settings
  cause difficulties beyond ~5000 features).
- When the input is raw vendor data or an mzML file — specXplore currently requires
  a preprocessed .mgf file with a 'feature_id' key; feature lists without a valid
  identifier cannot be used until renamed.
- When an unsupervised global clustering (e.g. k-medoid) assignment is the deliverable
  rather than a node-neighbor adjacency; the ego view is a per-node overlay, not a
  partition.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: specXplore
  role: Dashboard and importing pipeline that holds the precomputed ms2deepscore similarity
    matrix, renders the t-SNE overview, applies threshold and connectivity-limitation
    settings on-demand, and emits the ego edge overlay plus ego_network_adjacency.csv.
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Provides the state-of-the-art pairwise spectral similarity scores used as
    the quantitative backbone; the thresholded ms2deepscore matrix is the edge source
    for the ego subgraph.
  repo: https://github.com/matchms/ms2deepscore
- name: matchms
  role: Imports and preprocesses the .mgf MS/MS spectral data, including assigning/renaming
    the feature_id metadata key that ties each spectrum to its node in the similarity
    matrix.
  repo: https://github.com/matchms/matchms
- name: dash
  role: Framework underlying the specXplore dashboard in which the ego-network edge
    overlay is interactively toggled on-demand.
  repo: https://github.com/plotly/dash
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:a8e16e41e47a8b2dc8ad1d4a376244aeb5ac81f0abe3f7cb217952865c29df44
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ego-network-subgraph-extraction

## Summary

Extracts the local ego-network (all nodes adjacent to a selected reference node) from an ms2deepscore pairwise similarity matrix, thresholded and connectivity-limited, as an edge overlay for the specXplore t-SNE dashboard. This enables localized connectivity exploration around a single LC-MS/MS feature of interest.

## When to use

Use when a single LC-MS/MS feature (node) in the t-SNE overview needs its local neighborhood examined — i.e., the user wants to see which features are adjacent under a given similarity threshold, and optionally expand that neighborhood by regenerating under new constraints.

## When NOT to use

- When the goal is a global molecular-network overview across the whole feature set — specXplore's t-SNE plus edge overlays are explicitly designed for localized, ego-centric exploration, not exhaustive network rendering (the paper warns that liberal settings cause difficulties beyond ~5000 features).
- When the input is raw vendor data or an mzML file — specXplore currently requires a preprocessed .mgf file with a 'feature_id' key; feature lists without a valid identifier cannot be used until renamed.
- When an unsupervised global clustering (e.g. k-medoid) assignment is the deliverable rather than a node-neighbor adjacency; the ego view is a per-node overlay, not a partition.

## Inputs

- specXplore session data object (produced by the importing pipeline from an .mgf MS/MS feature list)
- ms2deepscore pairwise spectral similarity matrix (feature identifier ↔ row/column mapping)
- User-selected ego node identifier (feature_id) from the t-SNE overview
- Threshold and connectivity limitation settings: similarity threshold, minimum fragmentation overlap, top-K neighbor limits, maximal molecular family size, hop distance

## Outputs

- ego_network_adjacency.csv — tabular adjacency structure containing the named reference node, neighbor feature identifiers, edge weights/scores, and hop distance from the ego node
- Interactive edge overlay visualization on the specXplore t-SNE map showing which nodes are adjacent to the selected feature

## How to apply

Start from the specXplore importing pipeline output: the ms2deepscore pairwise similarity matrix computed alongside modified cosine and spec2vec scores from an .mgf-converted MS/MS feature list. Binarize the similarity matrix into edges using user-adjustable settings — pairwise similarity threshold with corresponding minimum fragmentation overlap (e.g. the 0.7 default cited for pairwise similarity thresholds), per-node top-K neighbor limits, and maximal molecular family sizes — to avoid dense topology that would make the ego view unreadable. Select the single reference (ego) node on the t-SNE map and collect all edges connecting to it, then edges connecting to those neighbors if a larger hop distance is configured. Assemble the subgraph as an adjacency structure keyed on feature_id (the identifier key the .mgf file must carry) with edge weights and hop distance from the ego node. Because thresholds can be changed rapidly, the method supports iterative expansion of neighborhood size: regenerate the local view under new constraints and compare, rather than accepting one fixed cutoff.

## Related tools

- **specXplore** (Dashboard and importing pipeline that holds the precomputed ms2deepscore similarity matrix, renders the t-SNE overview, applies threshold and connectivity-limitation settings on-demand, and emits the ego edge overlay plus ego_network_adjacency.csv.) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Provides the state-of-the-art pairwise spectral similarity scores used as the quantitative backbone; the thresholded ms2deepscore matrix is the edge source for the ego subgraph.) — https://github.com/matchms/ms2deepscore
- **matchms** (Imports and preprocesses the .mgf MS/MS spectral data, including assigning/renaming the feature_id metadata key that ties each spectrum to its node in the similarity matrix.) — https://github.com/matchms/matchms
- **dash** (Framework underlying the specXplore dashboard in which the ego-network edge overlay is interactively toggled on-demand.) — https://github.com/plotly/dash

## Examples

```
conda activate specxplore_environment && jupyter-notebook notebooks/demo.ipynb  # open specXplore, select ego node on t-SNE, set similarity threshold ~0.7 & top-K, export ego_network_adjacency.csv
```

## Evaluation signals

- ego_network_adjacency.csv exists, parses as a table, and contains the named reference (ego) node as a record key.
- Every recorded edge in the output file can be exactly re-derived from the thresholded ms2deepscore similarity matrix at the stated threshold and hop distance — i.e. a direct re-extraction from the binarized matrix reproduces the output row-for-row.
- Each neighbor's ms2deepscore value meets the configured similarity threshold AND the corresponding minimum fragmentation overlap, and no node exceeds the configured top-K neighbor limit or maximal molecular family size.
- The paper reports no independent quantitative benchmark for this view, so the structural consistency check between output and thresholded input matrix is the acceptance criterion.
- On macOS arm64 systems, ms2deepscore predictions may be silently unreliable (issue 199) — verify against a system where ms2deepscore is known to agree with published results.

## Limitations

- ms2deepscore is only expected to work well for spectra/metabolites well covered by or related to its training data; no coverage check is reported, so ego edges may be unreliable for dissimilar compound classes.
- Scalability beyond ~5000 features is untested; the dashboard is anticipated to face difficulties for larger datasets and liberal (low-threshold, high-connectivity) settings.
- On macOS arm64, a known ms2deepscore issue produces similarity predictions that silently disagree with other systems, directly affecting the ego edge generation.
- No online hosted version of specXplore exists; the tool must be installed locally, and Windows support is secondary.
- No tool version numbers are stated, and no automatic guideline is given for choosing threshold/top-K settings beyond manual expansion.

## Evidence

- [results] ego-network view as edge overlay revealing adjacency to a selected node: [evidence span withheld — anchor status: not_found]
- [results] connectivity determined by similarity threshold settings user can adjust: [evidence span withheld — anchor status: not_found]
- [abstract] t-SNE overview as jumping board for local connectivity exploration: "provides a two dimensional t-SNE embedding as a jumping board for local connectivity exploration"
- [abstract] fast changes of threshold and connectivity limitation settings: "while allowing fast changes of threshold and connectivity limitation settings, providing flexibility in adjusting settings to suit the localized node environment being explored"
- [abstract] ms2deepscore as quantitative backbone: "SpecXplore makes use of of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone"
- [readme] mgf input requirement and feature_id key: [evidence span withheld — anchor status: external_doc]
- [readme] README: feature_id expected key: [evidence span withheld — anchor status: external_doc]
- [discussion] pairwise similarity thresholds with minimum fragmentation overlaps default 0.7: "the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps"
- [discussion] top-K neighbor limits and maximal molecular family sizes as edge filters: "top-K neighbor limits on individual nodes"
- [discussion] edges visualized on demand with user-modifiable filter settings: "avoided by only visualizing edges on demand, using stringent user-modifiable edge filter settings."
- [readme] macos arm64 ms2deepscore issue 199 makes predictions unreliable: [evidence span withheld — anchor status: external_doc]
- [discussion] scalability limit ~5000 features anticipated: "We expect that the dashboard will scale well to 5000 features but face difficulties for larger data sets and liberal settings."
- [results] no independent quantitative benchmark for this view — structural consistency check is acceptance criterion: [evidence span withheld — anchor status: not_found]
- [intro] careful expansion of neighborhood size via threshold changes: "the ability to quickly change thresholds to regenerate local views under the new constraints, allowing the careful expansion of neighborhood size"
- [readme] install via pip git+https for specXplore: [evidence span withheld — anchor status: external_doc]
- [discussion] ms2deepscore limited to training-data-covered metabolites: "it can only be expected to work well for spectra and metabolites well covered by or related to its training data"
- [methods] perplexity tuning range for t-SNE: "test a range of values for the perplexity tuning parameter (usually between 5 and 50) and select an appropriate value."
