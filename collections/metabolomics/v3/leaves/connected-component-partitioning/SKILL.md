---
name: connected-component-partitioning
description: Use when a gene-gene edge list yields a 0-1 adjacency matrix too large
  for weighted adjacency estimation and you need connected-component decomposition
  plus community detection capped at 1,000 genes per cluster.
when_to_use_negative:
- The network's connected components are all already ≤1,000 genes, so no community-detection
  re-partitioning is needed — plain component decomposition suffices.
- Your downstream method requires a single full adjacency matrix rather than the list-of-smaller-matrices
  (block-diagonal) format NetGSA's NetGSA function expects.
- You need topology-based pathway scores directly; this skill only prepares adjacency
  inputs for NetGSA, it does not perform the enrichment test itself.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: netgsa
  role: Provides prepareAdjMat, the reference implementation whose clustering-based
    block-diagonal adjacency estimation this skill reconstructs; its NetGSA function
    consumes the resulting list of adjacency matrices.
  repo: github.com/mikehellstern/netgsa
- name: igraph
  role: Used for connected-components decomposition and the six community-detection
    algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain)
    applied to components with >1,000 genes.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:b290ce09df208401b3a2f4c57c9fa34ba5f128f5f81b01f89436acdf80ac5908
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - igraph
  - netgsa
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# connected-component-partitioning

## Summary

Partition a large gene-gene adjacency matrix into connected components and, when components exceed 1,000 genes, sub-partition them via igraph community detection so per-cluster adjacency estimation (as in netgsa's prepareAdjMat) stays tractable before reassembling a block-diagonal matrix.

## When to use

Use when a gene-gene edge list yields a 0-1 adjacency matrix too large for weighted adjacency estimation and you need connected-component decomposition plus community detection capped at 1,000 genes per cluster.

## When NOT to use

- The network's connected components are all already ≤1,000 genes, so no community-detection re-partitioning is needed — plain component decomposition suffices.
- Your downstream method requires a single full adjacency matrix rather than the list-of-smaller-matrices (block-diagonal) format NetGSA's NetGSA function expects.
- You need topology-based pathway scores directly; this skill only prepares adjacency inputs for NetGSA, it does not perform the enrichment test itself.

## Inputs

- Gene-gene edge list (from an interaction-retrieval module, e.g. KEGG/Reactome/PantherDB-derived interactions)
- 0-1 adjacency matrix constructed from the edge list

## Outputs

- Block-diagonal weighted adjacency matrix per condition (saved as an RDS file)
- Selected community-detection algorithm name and its edge-loss value
- Per-cluster weighted adjacency matrices

## How to apply

Load the gene-gene edge list in R and build the 0-1 adjacency matrix, then split it into connected components using igraph's connected-components decomposition. For any component containing more than 1,000 genes, run six igraph community-detection algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain) and discard any algorithm that produces clusters exceeding the 1,000-gene cap. Among the surviving algorithms, select the one with the smallest edge loss, then estimate the weighted adjacency matrix per cluster and reassemble the per-cluster matrices into a block-diagonal matrix per condition. This mirrors netgsa's prepareAdjMat logic, whose output feeds NetGSA's requirement that the input A be a list of adjacency matrices across tested groups (block-diagonal structure per group). Validate the reassembled block-diagonal matrix against reference prepareAdjMat output from the netgsa package.

## Related tools

- **netgsa** (Provides prepareAdjMat, the reference implementation whose clustering-based block-diagonal adjacency estimation this skill reconstructs; its NetGSA function consumes the resulting list of adjacency matrices.) — github.com/mikehellstern/netgsa
- **igraph** (Used for connected-components decomposition and the six community-detection algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain) applied to components with >1,000 genes.)

## Examples

```
library(igraph); comps <- components(graph_from_adjacency_matrix(A, mode='undirected')); clusters <- cluster_louvain(induced.subgraph(g, comps$membership==k))
```

## Evaluation signals

- The saved block-diagonal matrix matches the reference prepareAdjMat output from the netgsa package for the same edge list.
- No cluster produced by the selected algorithm exceeds 1,000 genes.
- The selected community-detection algorithm has the smallest edge loss among algorithms meeting the 1,000-gene cap.
- The output RDS contains the block-diagonal matrix plus the algorithm name and edge-loss value as metadata.
- The resulting structure is a list of adjacency matrices across tested groups, as required by NetGSA's default input A.

## Limitations

- The 1,000-gene cluster cap and the smallest-edge-loss selection rule are heuristics; alternative partitions may be biologically plausible.
- The netgsa README notes compatibility checks between adjacency matrices and input data were removed for block-diagonal structures ('should be added later'), so mismatched variables may pass silently — filter the data matrix to genes belonging to at least one tested pathway before running NetGSA.
- Pathway test statistics may not require assembling the entire adjacency matrix, but for many pathways the README raises that repeatedly subsetting variables could be inefficient versus assembling the full matrix.

## Evidence

- [abstract] abstract: "we have overhauled NetGSA, an existing topology-based method, to provide a computationally-efficient user-friendly tool"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
