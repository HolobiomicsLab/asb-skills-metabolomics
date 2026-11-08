---
name: graphml-network-export
description: Use when a similarity-weighted molecular network (edge weight = spectral
  similarity score) has been partitioned into Louvain communities and each community
  must be exported as a pruned, connected GraphML subnetwork for interpretability.
when_to_use_negative:
- Input is raw mass-spectrometry data (e.g. mzML or spectra files) rather than an
  already-built, similarity-weighted edge list with community labels.
- The analysis goal is to preserve cycles or full edge multiplicity within communities
  rather than a cycle-free most-important-links backbone.
- The network was produced by conventional arbitrary-threshold pruning, which the
  article notes 'can fracture natural metabolite families'; re-exporting such a network
  does not recover the lost relationships MCN is designed to rescue.
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0092
tools:
- name: NetworkX
  role: Partitions the full molecular network into per-community subgraphs, computes
    nx.maximum_spanning_tree with the spectral similarity score as edge weight, and
    writes each subnetwork to GraphML.
  repo: https://github.com/Alexander0/molecular_communities
- name: pandas
  role: Loads the spectral-similarity weighted edge list and the Louvain community
    assignment table before graph construction.
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:803c929afde8139e54fe99affd4944369958fb721663f467fa210b7ae4587fc7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - NetworkX
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# graphml-network-export

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Exports per-community molecular subnetworks as GraphML files after maximum-weight-spanning-tree pruning, so pruned clusters remain connected, cycle-free, and interoperable with standard graph tooling. It preserves the strongest spectral-similarity links that arbitrary threshold-based pruning would discard.

## When to use

Use when a similarity-weighted molecular network (edge weight = spectral similarity score) has been partitioned into Louvain communities and each community must be exported as a pruned, connected GraphML subnetwork for interpretability.

## When NOT to use

- Input is raw mass-spectrometry data (e.g. mzML or spectra files) rather than an already-built, similarity-weighted edge list with community labels.
- The analysis goal is to preserve cycles or full edge multiplicity within communities rather than a cycle-free most-important-links backbone.
- The network was produced by conventional arbitrary-threshold pruning, which the article notes 'can fracture natural metabolite families'; re-exporting such a network does not recover the lost relationships MCN is designed to rescue.

## Inputs

- Spectral-similarity weighted edge list (loadable with pandas)
- Louvain community assignment table mapping metabolite nodes to community labels

## Outputs

- community_spanning_trees.graphml containing one exported spanning-tree subnetwork per community
- Summary table of per-community node and edge counts

## How to apply

Load the spectral-similarity weighted edge list and the Louvain community assignment table with pandas, then build per-community subgraphs in NetworkX using the community labels. For each subgraph, compute nx.maximum_spanning_tree with the spectral similarity score as the edge weight, so only the strongest similarities are retained while every node assigned to that community stays connected in a cycle-free tree. Export each resulting tree with nx.write_graphml to community_spanning_trees.graphml and record per-community node and edge counts in a summary table. Validate that each exported subnetwork is a true spanning tree of its parent community: edge count must equal node count minus 1, and the node set must exactly match the Louvain assignment.

## Related tools

- **NetworkX** (Partitions the full molecular network into per-community subgraphs, computes nx.maximum_spanning_tree with the spectral similarity score as edge weight, and writes each subnetwork to GraphML.) — https://github.com/Alexander0/molecular_communities
- **pandas** (Loads the spectral-similarity weighted edge list and the Louvain community assignment table before graph construction.) — https://github.com/Alexander0/molecular_communities

## Examples

```
import networkx as nx; T = nx.maximum_spanning_tree(G, weight='weight'); nx.write_graphml(T, 'community_spanning_trees.graphml')
```

## Evaluation signals

- Each exported subnetwork is a spanning tree of its parent community: edge count equals node count minus 1.
- Each exported subnetwork contains every node assigned to that community by Louvain (node set matches the assignment table exactly).
- All retained edges are the highest-similarity links; no cycles remain in any exported subnetwork.
- community_spanning_trees.graphml is well-formed GraphML and per-community node/edge counts in the summary table match the exported files.

## Limitations

- Spanning-tree pruning discards all non-tree edges, so lower-similarity alternative connections within a community are not represented in the export.
- The approach depends on prior Louvain community detection; it inherits any errors or fragmentation from that partitioning.
- The empty repository README provides no additional implementation details beyond the article workflow.

## Evidence

- [methods] MCN spanning-tree pruning: [evidence span withheld — anchor status: not_found]
- [abstract] arbitrary thresholds caveat: "using arbitrary similarity thresholds that can fracture natural metabolite families"
- [abstract] MCN connectivity optimization: "We present molecular community networking (MCN), that optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise"
