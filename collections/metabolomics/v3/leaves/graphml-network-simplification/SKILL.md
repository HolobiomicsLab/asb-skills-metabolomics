---
name: graphml-network-simplification
description: Use when a precomputed GNPS molecular network is available as a .graphml
  file (e.g. pnordicum_ms2_gnps.graphml) and you need clean connected components,
  cliques or pairs to interpret with fragmentation patterns.
when_to_use_negative:
- You only have raw MS/MS spectra (MGF) and no precomputed GNPS network yet — the
  skill requires the .graphml network as input
- You need to mine the fragmentation patterns themselves; that is the mineMS2 frequent-subgraph
  mining step, not network simplification
- The network must retain weighted multi-edges or self-edges, since simplification
  deliberately removes them
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_3318
tools:
- name: mineMS2
  role: Provides findGNPSComponents and ships the pnordicum_ms2_gnps.graphml dataset
    used for component extraction
  repo: https://github.com/odisce/mineMS2
- name: igraph
  role: R library used to read the .graphml GNPS network (read_graph) and simplify
    it (remove self-edges, convert to simple graph)
- name: GNPS
  role: Platform that generated the molecular network; each vertex is a spectrum and
    each edge carries a modified cosine similarity score
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:1e9e2a0c547a7d4c597d5f9af24d4561175ac75137da7e8fbc9f49c18ed02418
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - igraph
  - mineMS2
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# graphml-network-simplification

## Summary

Load a precomputed GNPS molecular network (.graphml), simplify it with igraph (drop self-edges, convert the undirected disconnected graph to a simple graph), and extract components, disjoint cliques and high-similarity pairs so mineMS2 patterns can be matched against interpretable network substructures.

## When to use

Use when a precomputed GNPS molecular network is available as a .graphml file (e.g. pnordicum_ms2_gnps.graphml) and you need clean connected components, cliques or pairs to interpret with fragmentation patterns.

## When NOT to use

- You only have raw MS/MS spectra (MGF) and no precomputed GNPS network yet — the skill requires the .graphml network as input
- You need to mine the fragmentation patterns themselves; that is the mineMS2 frequent-subgraph mining step, not network simplification
- The network must retain weighted multi-edges or self-edges, since simplification deliberately removes them

## Inputs

- Precomputed GNPS molecular network in GraphML format (e.g. pnordicum_ms2_gnps.graphml, shipped as a mineMS2 package dataset)
- Spectrum metadata for the network vertices (e.g. dataset/pnordicum_ms2_info.tsv from the mineMS2 package)
- Parameters: minSize (e.g. 3) and pairThreshold on edge similarity (e.g. 0.9)

## Outputs

- Simplified igraph graph object (self-edges removed, undirected disconnected GNPS graph converted to simple graph)
- Data frame listing each selected component (connected components, disjoint cliques, high-similarity pairs) and its member spectrum indices
- Tab-separated component table, e.g. gnps_components_pnordicum.tsv

## How to apply

Load the precomputed GNPS network file (e.g. pnordicum_ms2_gnps.graphml shipped as a mineMS2 package dataset) with igraph's read_graph, then simplify it by removing self-edges and converting the undirected, disconnected GNPS graph into a simple graph object. In the GNPS network each vertex is a spectrum and each edge carries a modified cosine similarity score, so simplification preserves the vertex set (the 51 Penicillium nordicum MS/MS spectra) while cleaning edge artifacts. Extract connected components and disjoint cliques with findGNPSComponents using minSize=3, and additionally select high-similarity spectrum pairs using pairThreshold=0.9 on the edge similarity attribute. Collate the selected components, cliques and pairs into a single data frame listing each component and its member spectrum indices. In the P. nordicum reference run this yields nine components: four connected components (clusters of 21 and 11 plus two pairs) and five disjoint cliques (three in the larger cluster, two in the smaller).

## Related tools

- **mineMS2** (Provides findGNPSComponents and ships the pnordicum_ms2_gnps.graphml dataset used for component extraction) — https://github.com/odisce/mineMS2
- **igraph** (R library used to read the .graphml GNPS network (read_graph) and simplify it (remove self-edges, convert to simple graph))
- **GNPS** (Platform that generated the molecular network; each vertex is a spectrum and each edge carries a modified cosine similarity score)

## Examples

```
library(igraph); g <- simplify(read_graph(system.file('dataset/pnordicum_ms2_gnps.graphml', package='mineMS2'), format='graphml'))
```

## Evaluation signals

- The simplified graph preserves the expected vertex set (51 P. nordicum MS/MS spectra) while self-edges are removed
- findGNPSComponents with minSize=3 and pairThreshold=0.9 returns exactly nine components for the P. nordicum reference network
- The nine components decompose as four connected components (clusters of 21 and 11 plus two pairs) and five disjoint cliques (three within the larger cluster, two within the smaller)
- The output TSV (gnps_components_pnordicum.tsv) contains one row per component with the member spectrum indices
- GNPS parameter values were chosen to obtain a network consistent with the one reported by the original authors (Supplementary File 1, Note S2)

## Limitations

- GNPS network parameters were selected ad hoc to obtain a network consistent with the one reported by the authors, so component counts are parameter-dependent
- GNPS molecular network visualization propagates annotations between pairs but cannot extract common structures shared by more than two spectra, which is why cliques/components must be extracted separately for pattern interpretation
- The reference workflow is validated only on the 51-spectrum Penicillium nordicum dataset shipped with the package

## Evidence

- [intro] In molecular networks, as generated by the GNPS platform, each vertex is a spectrum and an edge indicates a high pairwise similarity: "In molecular networks, as generated by the GNPS platform [27], each vertex is a spectrum and an edge indicates a high pairwise similarity"
- [intro] we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting: "we show the potential of coupling mineMS2 to molecular network approaches such as GNPS to help interpreting the detected components"
- [readme] The included dataset consists of 51 spectra from the untargeted study of the secondary metabolism of Penicillium nordicum: [evidence span withheld — anchor status: external_doc]
- [readme] mineMS2 can be further coupled to the GNPS MS/MS molecular networking methodology: [evidence span withheld — anchor status: external_doc]
