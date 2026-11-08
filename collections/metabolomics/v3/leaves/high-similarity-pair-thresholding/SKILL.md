---
name: high-similarity-pair-thresholding
description: Use when you have a precomputed GNPS molecular network (e.g. a .graphml
  file where vertices are MS/MS spectra and edges carry a pairwise modified-cosine
  similarity score) and need to isolate pairs of spectra whose similarity exceeds
  a chosen cutoff such as 0.9.
when_to_use_negative:
- Input is raw MS/MS spectra (e.g. an .mgf file) rather than an already computed GNPS
  molecular network — mine the spectra first with mineMS2 or GNPS before extracting
  components
- The goal is de novo fragmentation pattern mining from spectra, not selecting spectra
  subsets from an existing network — use the mineMS2 frequent-subgraph mining workflow
  (discretizeMzDifferences, graph building, subgraph mining) instead
- Probabilistic motif discovery (MS2LDA-style distributions of fragments and losses)
  is desired rather than exact graph-based similarity extraction from a GNPS network
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: mineMS2
  role: Provides findGNPSComponents and the packaged pnordicum_ms2_gnps.graphml dataset
    used for component and pair extraction
  repo: https://github.com/odisce/mineMS2
- name: R
  role: Host language in which findGNPSComponents, igraph graph handling and the TSV
    export are executed
- name: igraph
  role: Used to read the .graphml GNPS network and simplify it (removing self-edges,
    converting to a simple undirected graph) before component/pair extraction
- name: GNPS
  role: Source of the molecular network whose edges carry pairwise modified cosine
    similarity scores that the pairThreshold filters on
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-025-01051-y
    title: 'mineMS2: annotation of spectral libraries with exact fragmentation patterns'
  source_content_hash: sha256:e65fb8caa963bbe6c9ce4070f40d2b1aab301325f7f506d3f88e0896da88dceb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/odisce/mineMS2
  tools:
  - GNPS
  - R
  - igraph
  - mineMS2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# high-similarity-pair-thresholding

## Summary

Extracts high-similarity spectrum pairs from a precomputed GNPS molecular network by thresholding the edge similarity attribute (pairThreshold=0.9 in the mineMS2 Penicillium nordicum workflow), alongside connected components and disjoint cliques, using findGNPSComponents on a simplified igraph object.

## When to use

Use when you have a precomputed GNPS molecular network (e.g. a .graphml file where vertices are MS/MS spectra and edges carry a pairwise modified-cosine similarity score) and need to isolate pairs of spectra whose similarity exceeds a chosen cutoff such as 0.9.

## When NOT to use

- Input is raw MS/MS spectra (e.g. an .mgf file) rather than an already computed GNPS molecular network — mine the spectra first with mineMS2 or GNPS before extracting components
- The goal is de novo fragmentation pattern mining from spectra, not selecting spectra subsets from an existing network — use the mineMS2 frequent-subgraph mining workflow (discretizeMzDifferences, graph building, subgraph mining) instead
- Probabilistic motif discovery (MS2LDA-style distributions of fragments and losses) is desired rather than exact graph-based similarity extraction from a GNPS network

## Inputs

- Precomputed GNPS molecular network file in GraphML format (e.g. pnordicum_ms2_gnps.graphml shipped as a mineMS2 package dataset)
- pairThreshold value on the edge similarity attribute (0.9 in the P. nordicum workflow)
- minSize value for component/clique retention (3 in the P. nordicum workflow)

## Outputs

- Data frame listing each extracted component (connected components, disjoint cliques, high-similarity pairs) and its member spectrum indices
- Exported TSV table, e.g. gnps_components_pnordicum.tsv

## How to apply

Load the precomputed GNPS network file (e.g. the package dataset pnordicum_ms2_gnps.graphml) with igraph's read_graph, then remove self-edges and convert the undirected, disconnected GNPS graph to a simple graph. Call findGNPSComponents with minSize=3 (to retain connected components of at least three vertices plus disjoint cliques) and pairThreshold=0.9 applied to the edge similarity attribute, which selects spectrum pairs connected by very high modified-cosine similarity. Because the GNPS network is derived from the mineMS2-coupled analysis, edge similarities come from the GNPS modified cosine score, so a 0.9 threshold keeps only near-identical fragmentation behaviour. Collate the selected components, cliques and pairs into a single data frame listing each component type and its member spectrum indices, and export as a TSV (e.g. gnps_components_pnordicum.tsv). The GNPS network parameters should be selected so the reconstructed network is consistent with the one reported by the original authors.

## Related tools

- **mineMS2** (Provides findGNPSComponents and the packaged pnordicum_ms2_gnps.graphml dataset used for component and pair extraction) — https://github.com/odisce/mineMS2
- **R** (Host language in which findGNPSComponents, igraph graph handling and the TSV export are executed)
- **igraph** (Used to read the .graphml GNPS network and simplify it (removing self-edges, converting to a simple undirected graph) before component/pair extraction)
- **GNPS** (Source of the molecular network whose edges carry pairwise modified cosine similarity scores that the pairThreshold filters on)

## Examples

```
pnordicum.graph <- igraph::simplify(igraph::read_graph("pnordicum_ms2_gnps.graphml", format="graphml")); comps <- findGNPSComponents(pnordicum.graph, minSize=3, pairThreshold=0.9)
```

## Evaluation signals

- On the P. nordicum network, findGNPSComponents with minSize=3 and pairThreshold=0.9 yields nine components: four connected components (clusters of 21 and 11, plus two pairs) and five disjoint cliques (three within the larger cluster, two within the smaller cluster), plus the high-similarity pairs at threshold 0.9
- The output table (gnps_components_pnordicum.tsv) contains one row per component with its member spectrum indices and no missing indices relative to the input graph
- All retained pairs have edge similarity attribute values >= the pairThreshold (0.9) on the modified cosine score
- The simplified graph has no self-edges and is a simple undirected graph before component extraction
- The GNPS network was rebuilt with parameters consistent with those reported by the original authors (Supplementary File 1, Note S2), so component sizes match the published network

## Limitations

- GNPS molecular network visualization propagates annotations between pairs but cannot extract common structures shared by more than two spectra, so pair extraction alone does not reveal shared fragmentation structures
- The extracted pairs/cliques are only as good as the precomputed GNPS network; network parameters must be tuned to match the network reported by the original authors
- The threshold (0.9) and minSize (3) are dataset-specific choices; other datasets may require different cutoffs

## Evidence

- [readme] mineMS2 can be further coupled to the GNPS MS/MS molecular networking methodology (Watrous et al., 2012) to focus on patterns that best explain components: [evidence span withheld — anchor status: not_found]
- [intro] GNPS edge semantics: "In molecular networks, as generated by the GNPS platform [27], each vertex is a spectrum and an edge indicates a high pairwise similarity according to the modified cosine score"
- [results] network parameter consistency: "with selected parameter values to obtain the more consistent network with the one reported by the authors (see Supplementary File 1, Note S2)"
- [supplementary] dataset loading: "pnordicum_metadata.c <- system.file("dataset/pnordicum_ms2_info.tsv", package = "mineMS2")"
