---
name: spectral-index-component-membership-extraction
description: Use when you have a precomputed GNPS molecular network file (graphml)
  and need to collate which spectra belong to each connected component, clique, or
  high-similarity pair before linking them to fragmentation patterns.
when_to_use_negative:
- The network has not yet been computed — this skill works on a precomputed GNPS graphml
  network, not on raw LC-HRMS/MS2 files (use ms2Lib/mineMS2 pattern mining first,
  and GNPS for networking)
- You need to extract structures shared by more than two spectra from the raw spectra
  themselves — GNPS networking alone cannot do this; pair mineMS2 pattern mining with
  the extracted components instead
- Components of size 1–2 are the target without any threshold filtering — the default
  minSize=3 excludes singletons and only pairs above pairThreshold are retained
edam_topics: []
tools:
- name: mineMS2
  role: R package providing findGNPSComponents and the packaged pnordicum_ms2_gnps.graphml
    network dataset; used to partition the GNPS network into components, cliques,
    and pairs
  repo: https://github.com/odisce/mineMS2
- name: R
  role: Runtime environment; igraph read_graph is used to load and simplify the undirected,
    disconnected GNPS graph
- name: GNPS
  role: Upstream platform that generated the molecular network (vertices = spectra,
    edges = modified cosine similarity) that findGNPSComponents consumes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:0b8a9f4542b2bce1000f1e6637f57dc300e698644e3ee2b6db7dda7cfcb37ba2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - mineMS2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-index-component-membership-extraction

## Summary

Extracts component membership from a precomputed GNPS molecular network (.graphml) using mineMS2's findGNPSComponents, partitioning spectra into connected components (≥ minSize vertices), disjoint cliques, and high-similarity pairs above a cosine-similarity pairThreshold.

## When to use

Use when you have a precomputed GNPS molecular network file (graphml) and need to collate which spectra belong to each connected component, clique, or high-similarity pair before linking them to fragmentation patterns.

## When NOT to use

- The network has not yet been computed — this skill works on a precomputed GNPS graphml network, not on raw LC-HRMS/MS2 files (use ms2Lib/mineMS2 pattern mining first, and GNPS for networking)
- You need to extract structures shared by more than two spectra from the raw spectra themselves — GNPS networking alone cannot do this; pair mineMS2 pattern mining with the extracted components instead
- Components of size 1–2 are the target without any threshold filtering — the default minSize=3 excludes singletons and only pairs above pairThreshold are retained

## Inputs

- Precomputed GNPS molecular network in GraphML format (e.g. pnordicum_ms2_gnps.graphml) with edge similarity attributes (modified cosine score)
- Spectrum metadata table (e.g. dataset/pnordicum_ms2_info.tsv) mapping spectrum indices to annotations
- Parameters: minSize (e.g. 3) and pairThreshold on edge similarity (e.g. 0.9)

## Outputs

- Data frame / TSV (e.g. gnps_components_pnordicum.tsv) listing each component (connected component, disjoint clique, or high-similarity pair) and its member spectrum indices

## How to apply

Load the precomputed GNPS network file (e.g. pnordicum_ms2_gnps.graphml, shipped as a mineMS2 package dataset) with the igraph read_graph function, then simplify it by removing self-edges and converting the undirected, disconnected GNPS graph to a simple graph object. Call findGNPSComponents filtering for components with at least minSize=3 vertices plus disjoint cliques, and extract high-similarity spectrum pairs with pairThreshold=0.9 applied to the edge similarity attribute (modified cosine score from GNPS). On the P. nordicum network (51 spectra from the secondary-metabolism study), this configuration yields nine components: four connected components (clusters of 21 and 11 plus two pairs) and five disjoint cliques (three within the larger cluster, two within the smaller one). Collate the selected components, cliques, and pairs into a single data frame listing each component and its member spectrum indices (e.g. gnps_components_pnordicum.tsv) so they can subsequently be matched against mined fragmentation patterns.

## Related tools

- **mineMS2** (R package providing findGNPSComponents and the packaged pnordicum_ms2_gnps.graphml network dataset; used to partition the GNPS network into components, cliques, and pairs) — https://github.com/odisce/mineMS2
- **R** (Runtime environment; igraph read_graph is used to load and simplify the undirected, disconnected GNPS graph)
- **GNPS** (Upstream platform that generated the molecular network (vertices = spectra, edges = modified cosine similarity) that findGNPSComponents consumes)

## Examples

```
gnps.graph <- read_graph(system.file("dataset/pnordicum_ms2_gnps.graphml", package="mineMS2"), format="graphml"); comps <- findGNPSComponents(simplify(gnps.graph), minSize=3, pairThreshold=0.9)
```

## Evaluation signals

- The expected yield on the P. nordicum network with minSize=3, pairThreshold=0.9 is exactly nine components: four connected components (sizes 21 and 11 plus two pairs) and five disjoint cliques (three in the 21-node cluster, two in the 11-node cluster)
- The output table contains one row per retained component with its member spectrum indices and can be written as gnps_components_pnordicum.tsv
- The graph was simplified before extraction: no self-edges remain and the undirected, disconnected GNPS graph is a simple graph object
- All retained pairs have edge similarity (modified cosine score) ≥ the pairThreshold of 0.9
- The number of distinct spectrum indices across components should be consistent with the 51 spectra in the Penicillium nordicum dataset

## Limitations

- GNPS molecular network visualization propagates annotations between pairs but cannot extract common structures shared by more than two spectra — component extraction must be combined with pattern mining (e.g. mineMS2) for interpretation
- Results depend on the upstream GNPS network parameters, which in the paper were selected to obtain a network consistent with the one reported by the original authors (Hautbergue et al. 2019)
- minSize=3 and pairThreshold=0.9 are tuned choices for the P. nordicum network; other networks may need different thresholds

## Evidence

- [other] findGNPSComponents on the P. nordicum network yields nine components: four connected components (two clusters of 21 and 11, plus two pairs) and five disjoint cliques (three within the larger cluster, two within the smaller cluster).: [evidence span withheld — anchor status: not_found]
- [other] Load the precomputed GNPS network file pnordicum_ms2_gnps.graphml (shipped as a package dataset) using the igraph read_graph function.: [evidence span withheld — anchor status: not_found]
- [readme] The included dataset, which is used in the examples and vignettes, consists of 51 spectra from the untargeted study of the secondary metabolism of Penicillium nordicum: [evidence span withheld — anchor status: not_found]
- [intro] In molecular networks, as generated by the GNPS platform, each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score: "In molecular networks, as generated by the GNPS platform [27], each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score"
- [supplementary] pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2"): "pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2")"
