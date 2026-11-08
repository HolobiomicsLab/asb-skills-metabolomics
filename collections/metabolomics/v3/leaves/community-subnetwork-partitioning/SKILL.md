---
name: community-subnetwork-partitioning
description: Use when a weighted spectral-similarity molecular network with Louvain
  community assignments needs to be split into interpretable, cycle-free community
  subnetworks without dropping any metabolite nodes.
when_to_use_negative:
- The goal is global clustering or community detection itself — this skill assumes
  Louvain community labels already exist and only prunes/exports subnetworks.
- You need to keep all edges (e.g. full weighted network analysis); spanning-tree
  pruning discards cycle-forming lower-weight edges by design.
- The input network is unweighted or lacks a similarity score usable as an edge weight,
  since maximum-spanning-tree pruning depends on edge weights.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NetworkX
  role: Used to partition the full molecular network into per-community subgraphs
    and compute each community's maximum weight spanning tree (networkx.maximum_spanning_tree)
    with spectral similarity as edge weight.
- name: pandas
  role: Loads the weighted edge list and the Louvain community assignment table before
    subgraph extraction.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:be6db3d09f660bf025eec28823b6bdc2b91a9570296f47e896252a016c4772bd
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NetworkX
  - pandas
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# community-subnetwork-partitioning

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Partitions a Louvain-community-labeled molecular network into per-community subgraphs and reduces each to its maximum-weight spanning tree (edge weight = spectral similarity), preserving full node connectivity while discarding redundant edges for interpretability.

## When to use

Use when a weighted spectral-similarity molecular network with Louvain community assignments needs to be split into interpretable, cycle-free community subnetworks without dropping any metabolite nodes.

## When NOT to use

- The goal is global clustering or community detection itself — this skill assumes Louvain community labels already exist and only prunes/exports subnetworks.
- You need to keep all edges (e.g. full weighted network analysis); spanning-tree pruning discards cycle-forming lower-weight edges by design.
- The input network is unweighted or lacks a similarity score usable as an edge weight, since maximum-spanning-tree pruning depends on edge weights.

## Inputs

- Spectral-similarity weighted edge list for the full molecular network (e.g. pandas-readable CSV/edge list)
- Louvain community assignment table mapping each molecular feature (node) to its community label

## Outputs

- community_spanning_trees.graphml containing one maximum-weight spanning-tree subnetwork per Louvain community
- Summary table of per-community node counts and edge counts

## How to apply

Load the spectral-similarity weighted edge list and the Louvain community assignment table with pandas, then partition the full molecular network into per-community subgraphs using networkx. For each community subgraph, compute the maximum weight spanning tree via networkx.maximum_spanning_tree, treating the spectral similarity score as the edge weight so that only the most important links remain while every node in the community stays connected. Export each resulting subnetwork to community_spanning_trees.graphml and record per-community node and edge counts in a summary table. The rationale is that MCN replaces arbitrary similarity thresholds — which can fracture natural metabolite families — with an optimization of connectivity for each metabolite, rescuing lost relationships and hidden metabolite connections.

## Related tools

- **NetworkX** (Used to partition the full molecular network into per-community subgraphs and compute each community's maximum weight spanning tree (networkx.maximum_spanning_tree) with spectral similarity as edge weight.)
- **pandas** (Loads the weighted edge list and the Louvain community assignment table before subgraph extraction.)

## Examples

```
import networkx as nx, pandas as pd
T = nx.maximum_spanning_tree(G.subgraph(nodes_of_community), weight='similarity_score')
nx.write_graphml(T, 'community_spanning_trees.graphml')
```

## Evaluation signals

- Each exported community subnetwork is a spanning tree: edge count equals node count minus 1.
- Each community subnetwork contains every node assigned to that community by Louvain (no dropped nodes).
- Per-community node and edge counts in the summary table match the graphml exports.
- Retained edges within each subnetwork are those with the highest spectral similarity scores, consistent with maximum-weight pruning.

## Limitations

- Spanning-tree pruning removes all cycle-forming edges, so secondary lower-weight similarity relationships within a community are not preserved in the exported subnetworks.
- Interpretability relies on Louvain community detection quality; fractured or misassigned communities carry into the pruned subnetworks.
- Unlike arbitrary threshold-based pruning, the method optimizes connectivity per metabolite but does not guarantee retention of every biologically meaningful similarity edge.

## Evidence

- [other] MCN treats each Louvain-detected community as a separate network and computes its maximum weight spanning tree (edge weight = similarity score), yielding cycle-free subnetworks that connect all nodes using only the most important links.: [evidence span withheld — anchor status: not_found]
- [abstract] using arbitrary similarity thresholds that can fracture natural metabolite families: "using arbitrary similarity thresholds that can fracture natural metabolite families"
- [abstract] We present molecular community networking (MCN), that optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise "hidden" metabolite connections.: "optimizes connectivity for each metabolite, rescuing lost relationships and capturing otherwise "hidden" metabolite connections"
