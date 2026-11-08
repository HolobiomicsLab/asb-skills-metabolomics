---
name: adjacency-matrix-construction
description: Use when building pathway-level adjacency matrices from gene-gene interaction
  edges for NetGSA, especially when networks are large (>1,000 genes per component)
  and block-diagonal per-condition structure is needed.
when_to_use_negative:
- When the network is small enough that every connected component already has ≤1,000
  genes — clustering adds overhead without benefit
- When a user already supplies precomputed adjacency matrices with the required block-diagonal
  structure, making prepareAdjMat reconstruction unnecessary
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: netgsa
  role: Provides prepareAdjMat to build and cluster pathway adjacency matrices and
    NetGSA to consume them
  repo: github.com/mikehellstern/netgsa
- name: igraph
  role: Connected-components decomposition and community-detection algorithms (walktrap,
    leading_eigen, fast_greedy, label_prop, infomap, louvain)
- name: KEGG
  role: Source database of gene-gene interaction edges
- name: Reactome
  role: Source database of gene-gene interaction edges
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:d160b7d94a520f11dec16c46496ad30ced2485f73733e1e56d522b19baee90de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KEGG
  - Reactome
  - igraph
  - knitr
  - netgsa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# adjacency-matrix-construction

## Summary

Constructs gene-gene network adjacency matrices for netgsa topology-based pathway enrichment by curating interaction edges from databases (KEGG, Reactome), clustering large components, and assembling per-group block-diagonal weighted adjacency matrices.

## When to use

Use when building pathway-level adjacency matrices from gene-gene interaction edges for NetGSA, especially when networks are large (>1,000 genes per component) and block-diagonal per-condition structure is needed.

## When NOT to use

- When the network is small enough that every connected component already has ≤1,000 genes — clustering adds overhead without benefit
- When a user already supplies precomputed adjacency matrices with the required block-diagonal structure, making prepareAdjMat reconstruction unnecessary

## Inputs

- Gene-gene interaction edge list curated from KEGG/Reactome/PantherDB (obtainEdgeList output)
- Gene expression data matrix with genes belonging to at least one tested pathway
- Pathway membership annotation (e.g., 114 KEGG pathways)

## Outputs

- Block-diagonal weighted adjacency matrix per condition, saved as an RDS file
- Selected community-detection algorithm name and edge loss value
- List of per-group adjacency matrices accepted as the `A` argument of NetGSA

## How to apply

Load the curated gene-gene edge list (e.g., from KEGG or Reactome via obtainEdgeList) and build a 0-1 adjacency matrix. Split it into connected components with igraph's connected-components decomposition. For any component exceeding 1,000 genes, run six igraph community-detection algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain); discard algorithms that yield clusters >1,000 genes and select, among the rest, the one with the smallest edge loss. Estimate weighted adjacency matrices per cluster and reassemble them into a block-diagonal matrix per condition (the matrices across groups are not assumed to share block structure). Save the result as an RDS file together with the selected algorithm name and edge loss value for later verification.

## Related tools

- **netgsa** (Provides prepareAdjMat to build and cluster pathway adjacency matrices and NetGSA to consume them) — github.com/mikehellstern/netgsa
- **igraph** (Connected-components decomposition and community-detection algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain))
- **KEGG** (Source database of gene-gene interaction edges)
- **Reactome** (Source database of gene-gene interaction edges)

## Examples

```
A <- prepareAdjMat(X, pathways, interactome, cluster = TRUE)
```

## Evaluation signals

- Saved block-diagonal matrix matches the reference prepareAdjMat output from the netgsa package
- No cluster produced by the selected algorithm exceeds 1,000 genes
- The recorded algorithm-selection record reports the smallest edge loss among admissible algorithms
- Eigenvalue list (adj2inf) is unchanged for block-diagonal adjacency matrices, providing a consistency check
- Downstream NetGSA run (REHE, sampling with sample_n=0.25, sample_p=0.25) completes in minutes on ordinary hardware

## Limitations

- Algorithms yielding any cluster >1,000 genes are discarded, so on dense networks all six candidates may be inadmissible
- The NetGSA input check on compatibility between adjacency matrices and input data was removed for block-diagonal structure, so mismatches may pass silently
- Variables in the input data matrix should be filtered to keep only those belonging to at least one tested pathway before running NetGSA

## Evidence

- [other] cluster-size filter: [evidence span withheld — anchor status: not_found]
- [other] connected-components split: [evidence span withheld — anchor status: not_found]
- [other] block-diagonal output: [evidence span withheld — anchor status: not_found]
- [readme] NetGSA input structure: [evidence span withheld — anchor status: external_doc]
- [abstract] database curation: "removes the need for expert knowledge by directly curating gene-gene interaction information from multiple external databases"
