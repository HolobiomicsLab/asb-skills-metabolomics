---
name: maximum-spanning-tree-extraction
description: Use when a weighted molecular network has been partitioned into Louvain
  communities and each cluster must be reduced to an interpretable, fully connected
  subnetwork without dropping any member nodes.
when_to_use_negative:
- The goal is to threshold the full molecular network globally by a similarity cutoff
  rather than preserve all nodes per community — the article states arbitrary similarity
  thresholds can fracture natural metabolite families
- Input is an unweighted graph, since the spanning tree must be maximized on the spectral-similarity
  edge weight
- Community partitions have not yet been computed, as this skill operates on per-community
  subgraphs
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: NetworkX
  role: Builds per-community subgraphs and computes each community's maximum weight
    spanning tree with networkx.maximum_spanning_tree
  repo: https://github.com/Alexander0/molecular_communities
- name: pandas
  role: Loads the spectral-similarity weighted edge list and the Louvain community
    assignment table, and records the per-community summary table
  repo: https://github.com/Alexander0/molecular_communities
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:394e7adfd8560b663ecb29f2255fd2e84ea3823b5793f6b3a408837b09b2632a
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Alexander0/molecular_communities
  tools:
  - NetworkX
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# maximum-spanning-tree-extraction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Prunes each Louvain-detected molecular community into a cycle-free maximum-weight spanning tree, retaining the strongest spectral-similarity edges while keeping all community nodes connected. This replaces arbitrary similarity-threshold cutoffs that fracture natural metabolite families.

## When to use

Use when a weighted molecular network has been partitioned into Louvain communities and each cluster must be reduced to an interpretable, fully connected subnetwork without dropping any member nodes.

## When NOT to use

- The goal is to threshold the full molecular network globally by a similarity cutoff rather than preserve all nodes per community — the article states arbitrary similarity thresholds can fracture natural metabolite families
- Input is an unweighted graph, since the spanning tree must be maximized on the spectral-similarity edge weight
- Community partitions have not yet been computed, as this skill operates on per-community subgraphs

## Inputs

- Spectral-similarity weighted edge list for the molecular network
- Louvain community assignment table mapping each metabolite node to its community
- Per-community subgraphs built from the full network using community labels

## Outputs

- One cycle-free maximum-weight spanning tree per community, exported as community_spanning_trees.graphml
- Summary table of per-community node and edge counts

## How to apply

Load the spectral-similarity weighted edge list and the Louvain community assignment table with pandas, then split the full molecular network into per-community subgraphs with networkx using the community labels. For each subgraph, call networkx.maximum_spanning_tree with the spectral similarity score as the edge weight so only the most important links are retained while every assigned node remains connected. Export each resulting tree (e.g. to community_spanning_trees.graphml) and record per-community node and edge counts in a summary table. The rationale is that MCN optimizes connectivity for each metabolite rather than applying arbitrary similarity thresholds, which can fracture natural metabolite families and lose hidden connections.

## Related tools

- **NetworkX** (Builds per-community subgraphs and computes each community's maximum weight spanning tree with networkx.maximum_spanning_tree) — https://github.com/Alexander0/molecular_communities
- **pandas** (Loads the spectral-similarity weighted edge list and the Louvain community assignment table, and records the per-community summary table) — https://github.com/Alexander0/molecular_communities

## Examples

```
import pandas as pd, networkx as nx; G = nx.from_pandas_edgelist(df, 'node_a', 'node_b', edge_attr='score'); T = nx.maximum_spanning_tree(G, weight='score'); nx.write_graphml(T, 'community_spanning_trees.graphml')
```

## Evaluation signals

- Every exported subnetwork is a spanning tree: edge count equals node count minus 1
- Each subnetwork contains every node assigned to that community by Louvain — no member nodes dropped
- Retained edges are the highest spectral-similarity links, so no weak-threshold edges displace stronger ones
- The number of exported subnetworks matches the number of Louvain communities
- community_spanning_trees.graphml parses correctly and per-community counts in the summary table match the exported graphs

## Limitations

- Spanning trees are necessarily cycle-free, so alternative similarity relationships within a community (parallel paths between nodes) are discarded
- Pruning depends on the prior Louvain partition; poorly detected communities yield poorly pruned trees
- Edge weights must be reliable spectral similarity scores — the tree optimization is only as good as the similarity metric

## Evidence

- [abstract] Arbitrary thresholds fracture metabolite families: [evidence span withheld — anchor status: not_found]
- [methods] Max-weight spanning tree per community: [evidence span withheld — anchor status: not_found]
- [methods] Validation invariant for exported trees: [evidence span withheld — anchor status: not_found]
