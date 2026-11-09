---
name: mass-spectral-feature-connectivity-analysis
description: Use when investigating which LC-MS/MS features are adjacent to one selected
  feature in a pairwise spectral similarity matrix, and you need to expand that neighborhood
  interactively by adjusting similarity thresholds and connectivity limits.
when_to_use_negative:
- Input is raw vendor data without a precomputed pairwise similarity matrix — the
  skill requires the ms2deepscore similarity backbone from the specXplore importing
  pipeline, not raw spectra files
- Goal is a global network of the whole dataset (e.g. a full molecular network on
  GNPS servers) rather than a localized ego-network around one node — specXplore was
  created with adjustable settings for local network topology in mind, not to replace
  traditional molecular networking
- Dataset contains more than ~5000 features; the article anticipates dashboard difficulties
  for larger data sets and liberal settings
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: specXplore
  role: Interactive Python dashboard in which the ego-network edge overlay on the
    t-SNE embedding is rendered and thresholds are adjusted on demand
  repo: https://github.com/kevinmildau/specXplore
- name: ms2deepscore
  role: Provides the state-of-the-art pairwise spectral similarity scores that serve
    as the quantitative backbone for edge binarization
  repo: https://github.com/matchms/ms2deepscore
- name: dash
  role: Framework underlying the specXplore dashboard for interactive local-connectivity
    exploration
- name: matchms
  role: Imports the .mgf spectral data and supplies the Spectrum module used to ensure
    the feature_id key required by the importing pipeline
  repo: https://matchms.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c04444
    title: Tailored Mass Spectral Data Exploration Using the SpecXplore Interactive
      Dashboard
  source_content_hash: sha256:cc68d779093f50814c67e28b81ab106e6183eebdf46df4c052b2a2f6305caae0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mass-spectral-feature-connectivity-analysis

## Summary

Explores local connectivity around a single LC-MS/MS feature of interest by overlaying an ego-network of edges derived from a thresholded ms2deepscore pairwise similarity matrix onto a t-SNE overview, as implemented in the specXplore interactive Python dashboard.

## When to use

Use when investigating which LC-MS/MS features are adjacent to one selected feature in a pairwise spectral similarity matrix, and you need to expand that neighborhood interactively by adjusting similarity thresholds and connectivity limits.

## When NOT to use

- Input is raw vendor data without a precomputed pairwise similarity matrix — the skill requires the ms2deepscore similarity backbone from the specXplore importing pipeline, not raw spectra files
- Goal is a global network of the whole dataset (e.g. a full molecular network on GNPS servers) rather than a localized ego-network around one node — specXplore was created with adjustable settings for local network topology in mind, not to replace traditional molecular networking
- Dataset contains more than ~5000 features; the article anticipates dashboard difficulties for larger data sets and liberal settings

## Inputs

- Precomputed ms2deepscore pairwise similarity matrix for an LC-MS/MS feature set (feature_id to matrix row/column mapping)
- Selected single reference (ego) feature node on the t-SNE embedding
- User-adjustable similarity threshold, minimum fragmentation overlap, top-K neighbor limit, maximal molecular family size, and hop distance settings

## Outputs

- Ego-network adjacency structure (reference node, neighbor feature identifiers, edge weights/scores, hop distance from ego)
- Serialized ego sub-network file, e.g. ego_network_adjacency.csv
- Interactive partial network drawing (edge overlay) on the t-SNE overview in the specXplore dashboard

## How to apply

Load the precomputed ms2deepscore pairwise similarity matrix for the LC-MS/MS feature set (feature identifier to row/column mapping) into the specXplore dashboard. Binarize the similarity matrix into edges using a user-adjustable similarity threshold (e.g. the 0.7 default cited for pairwise similarity thresholds with minimum fragmentation overlaps), combined with per-node top-K neighbor limits and maximal molecular family sizes to avoid dense topology. Select a single reference (ego) node on the t-SNE embedding and extract all edges connecting to it, then all edges connecting to those neighbors, iterating outward in line with the configured hop distance. Assemble the extracted subgraph as an adjacency structure (reference node, neighbor identifiers, edge weights/scores, hop distance from ego) and serialize it, e.g. to ego_network_adjacency.csv. Rationale: the article stresses that the t-SNE overview alone is difficult to read; local connectivity exploration via partial network drawings is the intended complement, and fast threshold changes allow careful expansion of the neighborhood size around a node of interest.

## Related tools

- **specXplore** (Interactive Python dashboard in which the ego-network edge overlay on the t-SNE embedding is rendered and thresholds are adjusted on demand) — https://github.com/kevinmildau/specXplore
- **ms2deepscore** (Provides the state-of-the-art pairwise spectral similarity scores that serve as the quantitative backbone for edge binarization) — https://github.com/matchms/ms2deepscore
- **dash** (Framework underlying the specXplore dashboard for interactive local-connectivity exploration)
- **matchms** (Imports the .mgf spectral data and supplies the Spectrum module used to ensure the feature_id key required by the importing pipeline) — https://matchms.readthedocs.io/en/latest/

## Examples

```
conda activate specxplore_environment && jupyter-notebook notebooks/demo.ipynb  # then in the dashboard: select node, adjust similarity threshold & top-K, export ego_network_adjacency.csv
```

## Evaluation signals

- The output ego_network_adjacency.csv exists, parses as a table, and contains the named reference node
- Recorded edges in the serialized ego sub-network exactly match a direct re-extraction from the thresholded ms2deepscore similarity matrix at the stated threshold and hop distance
- Every edge weight/score in the ego sub-network is at or above the configured similarity threshold (e.g. 0.7) and satisfies the minimum fragmentation overlap constraint
- Number of edges incident to the ego node does not exceed the configured top-K neighbor limit
- On the dashboard, the edge overlay around the selected node visually reflects only the nodes adjacent to it under the current threshold, and changing the threshold regenerates the local view accordingly

## Limitations

- ms2deepscore applicability is limited to spectra and metabolites well covered by or related to its training data; no coverage check is reported in the article
- Scalability beyond ~5000 features is untested; the article expects dashboard difficulties for larger data sets and liberal settings
- On macOS arm64 systems, a known ms2deepscore issue may make similarity predictions unreliable without any error or warning, directly affecting the specXplore importing pipeline and hence the similarity matrix this skill consumes
- specXplore currently fails on Windows; the article also notes no online hosted version is available
- No independent quantitative benchmark for the ego-network view is reported; acceptance is judged by structural consistency of the extracted subgraph against the thresholded matrix

## Evidence

- [results] edge overlay visualizations that reveal which nodes are adjacent to a selected node, with connectivity determined by similarity threshold settings the user can adjust: [evidence span withheld — anchor status: not_found]
- [intro] changing thresholds regenerates the local view under new constraints: "the ability to quickly change thresholds to regenerate local views under the new constraints, allowing the careful expansion of neighborhood size"
- [abstract] providing flexibility in adjusting settings to suit the localized node environment being explored: "while allowing fast changes of threshold and connectivity limitation settings, providing flexibility in adjusting settings to suit the localized node environment being explored"
- [discussion] the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps: "the pairwise similarity thresholds used as well as corresponding minimum fragmentation overlaps"
- [discussion] avoided by only visualizing edges on demand, using stringent user-modifiable edge filter settings: "avoided by only visualizing edges on demand, using stringent user-modifiable edge filter settings."
- [readme] specXplore expects the feature identifier key to be feature_id: [evidence span withheld — anchor status: external_doc]
- [abstract] state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone: "SpecXplore makes use of of state of the art ms2deepscore pairwise spectral similarities as a quantitative backbone"
