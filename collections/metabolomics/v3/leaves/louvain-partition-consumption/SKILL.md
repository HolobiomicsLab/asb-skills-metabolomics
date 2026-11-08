---
name: louvain-partition-consumption
description: Use when a molecular network already has Louvain community labels (e.g.,
  from a spectral-similarity edge list) and you need interpretable, cycle-free per-cluster
  subnetworks that preserve connectivity of every assigned metabolite.
when_to_use_negative:
- The analysis goal is conventional threshold-based molecular networking, where edges
  below an arbitrary similarity cutoff are simply removed rather than replaced by
  a spanning-tree backbone.
- The network has no precomputed Louvain partition, so there is no community structure
  to consume and per-community subgraphs cannot be formed.
- Retaining cycles within communities is essential (e.g., for downstream loop/alternating-path
  analyses), since spanning trees are by definition cycle-free.
edam_operation: http://edamontology.org/operation_0277
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: NetworkX
  role: Partition the full molecular network into per-community subgraphs and compute
    each community's maximum-weight spanning tree; write community_spanning_trees.graphml
- name: pandas
  role: Load the spectral-similarity edge list and Louvain community assignment table,
    and record the per-community node/edge count summary
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.02.606356
    title: Ordering molecular diversity in untargeted metabolomics via molecular community
      networking
  source_content_hash: sha256:4fc94e12e2124f3d8d894ccce2d6c80ed9a8b0d90a5c55fdb38fc14c4e963e67
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - NetworkX
  - pandas
  edam_operation: http://edamontology.org/operation_0277
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# louvain-partition-consumption

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Consumes the Louvain community assignment of a spectral-similarity-weighted molecular network by splitting it into per-community subgraphs and pruning each to its maximum-weight spanning tree, retaining the strongest similarity edges while keeping every community node connected. This avoids the arbitrary similarity-threshold pruning that the MCN paper argues fractures natural metabolite families.

## When to use

Use when a molecular network already has Louvain community labels (e.g., from a spectral-similarity edge list) and you need interpretable, cycle-free per-cluster subnetworks that preserve connectivity of every assigned metabolite.

## When NOT to use

- The analysis goal is conventional threshold-based molecular networking, where edges below an arbitrary similarity cutoff are simply removed rather than replaced by a spanning-tree backbone.
- The network has no precomputed Louvain partition, so there is no community structure to consume and per-community subgraphs cannot be formed.
- Retaining cycles within communities is essential (e.g., for downstream loop/alternating-path analyses), since spanning trees are by definition cycle-free.

## Inputs

- spectral-similarity weighted edge list (loadable with pandas)
- Louvain community assignment table mapping nodes to community labels

## Outputs

- community_spanning_trees.graphml: one maximum-weight spanning tree per Louvain community
- summary table of per-community node and edge counts

## How to apply

Load the spectral-similarity weighted edge list and the Louvain community assignment table with pandas. Partition the full molecular network with networkx by grouping nodes according to their community labels, producing one subgraph per Louvain community. For each subgraph, compute networkx.maximum_spanning_tree using the spectral similarity score as the edge weight, so only the strongest similarities are retained while all nodes remain connected. Export each tree to community_spanning_trees.graphml and record per-community node and edge counts in a summary table. The rationale is that MCN 'optimizes connectivity for each metabolite', so tree pruning recovers relationships that a fixed similarity cutoff would discard.

## Related tools

- **NetworkX** (Partition the full molecular network into per-community subgraphs and compute each community's maximum-weight spanning tree; write community_spanning_trees.graphml)
- **pandas** (Load the spectral-similarity edge list and Louvain community assignment table, and record the per-community node/edge count summary)

## Examples

```
T = nx.maximum_spanning_tree(G.subgraph(nodes_of_community), weight='similarity'); nx.write_graphml(T, 'community_spanning_trees.graphml')
```

## Evaluation signals

- Every exported community subnetwork is a spanning tree of its parent community: edge count equals node count minus 1.
- Each exported subnetwork contains every node assigned to that community by Louvain (no nodes dropped during pruning).
- Edge weights in each output tree are spectral similarity scores, and each retained edge is the strongest available connector in the community.
- The number of exported GraphML files equals the number of Louvain communities, and the summary table matches the GraphML node/edge counts.
- Cross-check against the naive-threshold baseline: relationships present in the full network but lost under an arbitrary cutoff should appear in the spanning trees.

## Limitations

- Pruning to a spanning tree removes cycles, so secondary similarity relationships within a community are discarded and only a cycle-free backbone survives.
- The result depends on the upstream Louvain partition; nodes misassigned by community detection will land in the wrong spanning tree.
- Arbitrary similarity thresholds, in contrast, can fracture natural metabolite families — but spanning-tree pruning still requires the community structure to be correct to be meaningful.

## Evidence

- [abstract] thresholds fracture families: [evidence span withheld — anchor status: not_found]
