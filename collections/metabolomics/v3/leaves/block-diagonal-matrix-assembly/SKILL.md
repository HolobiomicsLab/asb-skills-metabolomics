---
name: block-diagonal-matrix-assembly
description: Use when a gene-gene edge list yields a large 0-1 adjacency matrix (connected
  components >1,000 genes) that must be decomposed into cluster-wise weighted adjacency
  matrices and reassembled as a block-diagonal matrix for NetGSA input.
when_to_use_negative:
- Every connected component already contains ≤1,000 genes, so no clustering-based
  decomposition is needed
- The downstream method does not accept a block-diagonal adjacency structure and requires
  a single full p x p adjacency matrix
- Adjacency matrices across tested groups must share an identical block diagonal structure
  that this skill's per-condition reassembly does not guarantee
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0203
tools:
- name: netgsa
  role: Provides prepareAdjMat, whose clustering-based block-diagonal adjacency estimation
    this skill reconstructs; the resulting matrix is consumed by the NetGSA enrichment
    function
  repo: https://github.com/mikehellstern/netgsa
- name: igraph
  role: Performs connected-components decomposition and the six community-detection
    algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain)
    used to cluster genes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:22dd2d1540df5115e6763109f20b2914f2570eea8d2b3d46509a52c4e36988be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikehellstern/netgsa
  tools:
  - igraph
  - netgsa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# block-diagonal-matrix-assembly

## Summary

Reconstructs the prepareAdjMat step of netgsa: cluster large gene networks with igraph community-detection algorithms, estimate weighted adjacency matrices per cluster, and reassemble them into a block-diagonal adjacency matrix for each condition before NetGSA pathway enrichment.

## When to use

Use when a gene-gene edge list yields a large 0-1 adjacency matrix (connected components >1,000 genes) that must be decomposed into cluster-wise weighted adjacency matrices and reassembled as a block-diagonal matrix for NetGSA input.

## When NOT to use

- Every connected component already contains ≤1,000 genes, so no clustering-based decomposition is needed
- The downstream method does not accept a block-diagonal adjacency structure and requires a single full p x p adjacency matrix
- Adjacency matrices across tested groups must share an identical block diagonal structure that this skill's per-condition reassembly does not guarantee

## Inputs

- Gene-gene edge list produced by the netgsa retrieval module
- 0-1 adjacency matrix constructed from the edge list
-  igraph graph objects per connected component

## Outputs

- Block-diagonal weighted adjacency matrix per condition (saved as RDS)
- Selected community-detection algorithm name
- Edge loss value for the selected clustering
- List of cluster-wise weighted adjacency matrices

## How to apply

Load the gene-gene edge list in R and build the 0-1 adjacency matrix, then split it into connected components using igraph's connected-components decomposition. For any component with more than 1,000 genes, run six igraph community-detection algorithms: walktrap, leading_eigen, fast_greedy, label_prop, infomap, and louvain. Discard algorithms whose clusters exceed 1,000 genes, and among the remaining algorithms select the one with the smallest edge loss. Estimate the weighted adjacency matrices per cluster, reassemble them into a block-diagonal matrix for each condition, and save the result as an RDS file together with the selected algorithm name and edge loss value. The rationale is that clustering bounds the size of the matrix subproblems so NetGSA enrichment stays computationally efficient on a personal computer.

## Related tools

- **netgsa** (Provides prepareAdjMat, whose clustering-based block-diagonal adjacency estimation this skill reconstructs; the resulting matrix is consumed by the NetGSA enrichment function) — https://github.com/mikehellstern/netgsa
- **igraph** (Performs connected-components decomposition and the six community-detection algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain) used to cluster genes)

## Examples

```
A <- prepareAdjMat(X, group, pathways, edgelist); saveRDS(A$Adj, 'adj_blockdiag.rds')
```

## Evaluation signals

- Saved block-diagonal matrix matches the reference prepareAdjMat output from the netgsa package
- Algorithm-selection record in the RDS file names one of the six igraph algorithms and its edge loss value
- No cluster produced by the selected algorithm exceeds 1,000 genes
- Every algorithm yielding clusters >1,000 genes was discarded before selection
- Reassembled block-diagonal matrix covers exactly the genes present in at least one tested pathway

## Limitations

- Algorithms producing any cluster larger than 1,000 genes are discarded, so a dense component with no acceptable clustering cannot be processed
- The current implementation removed the compatibility check between adjacency matrices and input data for block-diagonal structure, which should be added later
- Adjacency matrices across groups are not assumed to share the same block diagonal structure, so per-condition matrices may differ
- Variables in the input data matrix should be filtered to keep only genes belonging to at least one tested pathway before running NetGSA

## Evidence

- [other] When clustering genes, netgsa discards algorithms producing a maximum cluster size >1,000 genes and, among the remaining algorithms, chooses the one with the smallest edge loss: [evidence span withheld — anchor status: not_found]
- [other] run the six igraph community-detection algorithms: [evidence span withheld — anchor status: not_found]
- [readme] README on block-diagonal adjacency structure in NetGSA input: [evidence span withheld — anchor status: external_doc]
- [readme] README on eigenvalue stability under block diagonal structure: [evidence span withheld — anchor status: external_doc]
- [abstract] Article abstract on computational efficiency motivating the clustering: "Pathway enrichment analysis for thousands of genes can be performed in minutes on a personal computer without sacrificing statistical power"
