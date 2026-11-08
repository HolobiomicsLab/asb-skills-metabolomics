---
name: community-detection-network-clustering
description: Use when a gene-gene interaction network contains connected components
  with more than 1,000 genes and weighted adjacency matrices must be estimated per
  pathway/condition for topology-based enrichment without exhaustive computation on
  the full graph.
when_to_use_negative:
- The network's connected components all contain 1,000 genes or fewer, so per-cluster
  estimation is unnecessary and the full adjacency matrix can be handled directly
- The analysis does not require weighted adjacency estimation (e.g., pathway-agnostic
  graph visualization where edge weights are irrelevant)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: netgsa
  role: R package whose prepareAdjMat step estimates weighted adjacency matrices and
    reassembles per-cluster matrices into a block-diagonal matrix
  repo: github.com/mikehellstern/netgsa
- name: igraph
  role: Used for connected-components decomposition of the adjacency matrix and for
    running the six community-detection algorithms (walktrap, leading_eigen, fast_greedy,
    label_prop, infomap, louvain)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:d1113198194b6e4962c0874903ee577bb5e1aef46ab040787bb4cf93b9c0d537
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - igraph
  - netgsa
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# community-detection-network-clustering

## Summary

Scales network-based adjacency estimation in netgsa by clustering large gene-gene networks into communities: six igraph community-detection algorithms are run on oversized connected components, the best algorithm is chosen by minimum edge loss, and weighted adjacency matrices are estimated per cluster and reassembled into a block-diagonal matrix.

## When to use

Use when a gene-gene interaction network contains connected components with more than 1,000 genes and weighted adjacency matrices must be estimated per pathway/condition for topology-based enrichment without exhaustive computation on the full graph.

## When NOT to use

- The network's connected components all contain 1,000 genes or fewer, so per-cluster estimation is unnecessary and the full adjacency matrix can be handled directly
- The analysis does not require weighted adjacency estimation (e.g., pathway-agnostic graph visualization where edge weights are irrelevant)

## Inputs

- Gene-gene edge list from the retrieval module (R data frame of gene-gene interactions)
- 0-1 adjacency matrix constructed from the edge list

## Outputs

- Block-diagonal weighted adjacency matrix per condition (saved as RDS)
- Selected community-detection algorithm name (RDS record)
- Edge loss value of the selected clustering (RDS record)

## How to apply

First load the gene-gene edge list into R and build the 0-1 adjacency matrix, then decompose it into connected components with igraph's connected-components decomposition. For every component exceeding 1,000 genes, run six igraph community-detection algorithms: walktrap, leading_eigen, fast_greedy, label_prop, infomap, and louvain. Discard any algorithm that produces a maximum cluster size above 1,000 genes; among the remaining algorithms select the one with the smallest edge loss (i.e., fewest within-network edges severed by the clustering). Estimate the weighted adjacency matrices separately for each cluster of the winning algorithm, then reassemble them into a block-diagonal matrix per condition and save it as an RDS file together with the selected algorithm name and its edge loss value.

## Related tools

- **netgsa** (R package whose prepareAdjMat step estimates weighted adjacency matrices and reassembles per-cluster matrices into a block-diagonal matrix) — github.com/mikehellstern/netgsa
- **igraph** (Used for connected-components decomposition of the adjacency matrix and for running the six community-detection algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain))

## Examples

```
Rscript -e 'library(netgsa); A <- prepareAdjMat(edgelist, genes, group.vec); saveRDS(A, "adjMat.rds")'
```

## Evaluation signals

- The block-diagonal matrix saved as RDS matches the reference prepareAdjMat output from the netgsa package
- The algorithm-selection record (chosen algorithm name and edge loss value) is consistent with the netgsa reference output
- No cluster produced by the chosen algorithm exceeds 1,000 genes
- The chosen algorithm has the smallest edge loss among all candidate algorithms that passed the 1,000-gene cluster-size filter
- The block-diagonal structure is reassembled correctly per condition, with each block corresponding to one cluster

## Limitations

- Algorithms yielding clusters larger than 1,000 genes are discarded, so if all six algorithms fail this filter no clustering is available for the component
- The NetGSA input adjacency matrix A is a list of per-group matrices and block-diagonal structures are not assumed to match across groups; a compatibility check between adjacency matrices and input data was removed and 'should be added later'
- Variables in the input data matrix should be filtered before running NetGSA to keep only those belonging to at least one tested pathway

## Evidence

- [full_text] When clustering genes, netgsa discards algorithms producing a maximum cluster size >1,000 genes and, among the remaining algorithms, chooses the one with the smallest edge loss: [evidence span withheld — anchor status: not_found]
- [full_text] weighted adjacency matrices are then estimated per cluster and reassembled into a block diagonal matrix: [evidence span withheld — anchor status: not_found]
- [readme] we assume that its adjacency matrix is again coded as a list of smaller matrices: [evidence span withheld — anchor status: external_doc]
- [readme] we should make sure to filter variables in the input data matrix to keep only those that belong to at least one tested pathway: [evidence span withheld — anchor status: external_doc]
