---
name: edge-loss-algorithm-selection
description: Use when estimating weighted adjacency matrices for pathway enrichment
  from a gene-gene edge list whose connected components exceed 1,000 genes, and a
  tractable block-diagonal adjacency structure is needed for NetGSA.
when_to_use_negative:
- The network has no connected component larger than 1,000 genes, so no clustering-based
  block decomposition is needed
- The adjacency matrices across groups are already provided as per-group lists of
  smaller matrices to NetGSA, bypassing the need to re-estimate a block-diagonal structure
- The goal is differential expression or non-topology-based enrichment (e.g. SPIA,
  PRS) that does not require an estimated adjacency matrix
edam_operation: http://edamontology.org/operation_3432
tools:
- name: netgsa
  role: Package whose prepareAdjMat routine performs the clustering, adjacency estimation,
    and block-diagonal reassembly
  repo: github.com/mikehellstern/netgsa
- name: igraph
  role: Provides connected-components decomposition and the six candidate community-detection
    algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1008979
    title: 'netgsa: Fast computation and interactive visualization for topology-based
      pathway enrichment analysis'
  source_content_hash: sha256:112a0b117c657758b894332aa77d8f41886b5fb446c289d8dc02cc0d908c5fc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - igraph
  - netgsa
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# edge-loss-algorithm-selection

## Summary

A clustering-based procedure from netgsa's prepareAdjMat that partitions large gene networks into blocks using igraph community-detection algorithms, selecting the algorithm with the smallest edge loss so that weighted adjacency matrices can be estimated per cluster and reassembled as a block-diagonal matrix.

## When to use

Use when estimating weighted adjacency matrices for pathway enrichment from a gene-gene edge list whose connected components exceed 1,000 genes, and a tractable block-diagonal adjacency structure is needed for NetGSA.

## When NOT to use

- The network has no connected component larger than 1,000 genes, so no clustering-based block decomposition is needed
- The adjacency matrices across groups are already provided as per-group lists of smaller matrices to NetGSA, bypassing the need to re-estimate a block-diagonal structure
- The goal is differential expression or non-topology-based enrichment (e.g. SPIA, PRS) that does not require an estimated adjacency matrix

## Inputs

- Gene-gene edge list from a retrieval module (R)
- 0-1 adjacency matrix constructed from the edge list
- Connected components of the adjacency matrix (igraph decomposition)

## Outputs

- Block-diagonal weighted adjacency matrix per condition (RDS file)
- Selected community-detection algorithm name
- Edge loss value for the selected algorithm

## How to apply

Construct the 0-1 adjacency matrix from the gene-gene edge list, then split it into connected components with igraph's connected-components decomposition. For any component with more than 1,000 genes, run six igraph community-detection algorithms: walktrap, leading_eigen, fast_greedy, label_prop, infomap, and louvain. Discard algorithms that produce clusters exceeding 1,000 genes; among the remaining algorithms, select the one with the smallest edge loss. Estimate the weighted adjacency matrices per selected cluster and reassemble them into a block-diagonal matrix per condition, saving it as an RDS file together with the chosen algorithm name and edge-loss value. This block structure matters because NetGSA accepts adjacency matrices coded as lists of smaller matrices per group, and eigenvalue lists remain unchanged under block-diagonal structure.

## Related tools

- **netgsa** (Package whose prepareAdjMat routine performs the clustering, adjacency estimation, and block-diagonal reassembly) — github.com/mikehellstern/netgsa
- **igraph** (Provides connected-components decomposition and the six candidate community-detection algorithms (walktrap, leading_eigen, fast_greedy, label_prop, infomap, louvain))

## Examples

```
devtools::install_github("mikehellstern/netgsa", build_vignettes=T)
```

## Evaluation signals

- No selected clustering yields a maximum cluster size greater than 1,000 genes
- Among non-discarded algorithms, the chosen algorithm has the smallest edge loss, and the edge-loss value is recorded with the output
- The saved block-diagonal matrix reconstructs, per condition, into a structure consistent with the reference prepareAdjMat output from the netgsa package
- The RDS output contains the block-diagonal matrix plus metadata: selected algorithm name and edge loss value
- The resulting adjacency can be supplied to NetGSA as a per-group list of smaller matrices without a variable-compatibility failure against the input data

## Limitations

- NetGSA no longer assumes adjacency matrices across groups share the same block-diagonal structure, and the check on variable compatibility between adjacency matrices and input data was removed and 'should be added later'
- The eigenvalue list via adj2inf is unchanged for block-diagonal adjacency, but assembling the entire adjacency matrix may be preferable when many pathways are tested to avoid repeatedly subsetting variables
- Variables in the input data matrix should be filtered to keep only those belonging to at least one tested pathway before running NetGSA

## Evidence

- [readme] README block-diagonal input convention: [evidence span withheld — anchor status: external_doc]
- [readme] README on adj2inf under block-diagonal structure: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract: overhauled NetGSA tool: "we have overhauled NetGSA, an existing topology-based method, to provide a computationally-efficient user-friendly tool"
- [intro] Intro: netgsa R package overhaul: "To address this need, we have completely overhauled the netgsa R package"
