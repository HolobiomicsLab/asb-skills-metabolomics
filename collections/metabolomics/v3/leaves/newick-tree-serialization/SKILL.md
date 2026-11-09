---
name: newick-tree-serialization
description: Use when a symmetric pairwise distance matrix over tandem MS datasets
  has been computed (e.g., by compareMS2) and a rooted, ultrametric tree in Newick
  format is the required output for phylogenetic display or downstream tooling.
when_to_use_negative:
- Input is raw tandem MS spectra or MGF files rather than an already-computed distance
  matrix — the pairwise comparison phase must run first
- A non-ultrametric or bootstrapped tree is required (e.g., neighbor-joining with
  support values); UPGMA assumes a constant rate and produces a rooted ultrametric
  tree without replicate support
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: compareMS2 2.0
  role: Computes the distance matrix and performs the UPGMA conversion to Newick in
    JavaScript; GUI renders the tree with phylotree.js
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: Independent tool used to verify the correct phylogenetic tree could be reconstructed
    from the same data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:e18ea03d21cff0552f3a4eb87bdd813c8957d81c985304b80b38385eefd0b1c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - MEGA11
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# newick-tree-serialization

## Summary

Converts a pairwise distance matrix from tandem MS dataset comparisons into a rooted Newick-format phylogenetic tree using UPGMA (implemented in JavaScript in compareMS2 2.0), so that molecular-phylogenetic relationships between LC-MS/MS datasets can be serialized and rendered.

## When to use

Use when a symmetric pairwise distance matrix over tandem MS datasets has been computed (e.g., by compareMS2) and a rooted, ultrametric tree in Newick format is the required output for phylogenetic display or downstream tooling.

## When NOT to use

- Input is raw tandem MS spectra or MGF files rather than an already-computed distance matrix — the pairwise comparison phase must run first
- A non-ultrametric or bootstrapped tree is required (e.g., neighbor-joining with support values); UPGMA assumes a constant rate and produces a rooted ultrametric tree without replicate support

## Inputs

- Symmetric pairwise distance matrix over LC-MS/MS (MGF) datasets, as computed by compareMS2's spectral similarity metric
- Optionally, a 'spectra_to_species.txt' sample-to-species mapping file (filename TAB species name per line) for labeling leaves

## Outputs

- Rooted Newick-format tree string with branch lengths after colons and a terminating semicolon
- Graphical UPGMA phylogenetic tree rendered in the compareMS2 2.0 GUI via phylotree.js, with a quality metric

## How to apply

Load the lower-triangular distance matrix produced by compareMS2's pairwise spectral comparisons and initialize each tandem MS dataset as a singleton leaf cluster at height 0. Iteratively merge the two clusters with the smallest pairwise distance, recomputing the merged cluster's distance to every remaining cluster as the size-weighted average of cross-cluster member distances (standard UPGMA). Set each merge node's height to half of the merged pair distance and derive branch lengths as parent height minus child height, which guarantees nonnegative ultrametric branch lengths. Serialize the resulting rooted hierarchy in Newick format with branch lengths after colons and a terminating semicolon; in compareMS2 2.0 this is done in JavaScript and the GUI renders the tree via phylotree.js, redrawing the UPGMA tree as each row of the distance matrix completes.

## Related tools

- **compareMS2 2.0** (Computes the distance matrix and performs the UPGMA conversion to Newick in JavaScript; GUI renders the tree with phylotree.js) — https://github.com/524D/compareMS2.git
- **MEGA11** (Independent tool used to verify the correct phylogenetic tree could be reconstructed from the same data)

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- Emitted Newick string is syntactically valid: balanced parentheses and a terminal semicolon
- All branch lengths are nonnegative, since each is derived as parent node height minus child node height
- The tree is ultrametric (all leaves equidistant from the root), as expected from UPGMA with heights set to half the merged pair distance
- Applied to the five serum datasets (42,629–47,626 spectra each), the topology matches the correct phylogenetic tree that the paper reports can be reconstructed in compareMS2 and MEGA11
- Tree topology is invariant to the order of rows/columns in the symmetric distance matrix (D(A,B) = D(B,A)) up to leaf-label ordering

## Limitations

- UPGMA produces a rooted ultrametric tree and assumes distances are compatible with a molecular clock; non-clock-like distance matrices yield distorted topologies
- The paper does not evaluate the matrix-to-Newick conversion in isolation, so validation relies on syntax checks and topological agreement with MEGA11
- Tree rendering updates only at the completion of each row of the lower-triangular distance matrix, so the displayed tree is provisional during long pairwise-comparison runs

## Evidence

- [readme] The compareMS2 GUI provides real-time feedback by continuously updating the distance matrix, and drawing a UPGMA tree at the completion of each row in the (lower triangular) distance matrix.: [evidence span withheld — anchor status: external_doc]
- [full_text] compareMS2 2.0 converts the distance matrix into Newick format using the UPGMA method, which is implemented in JavaScript, while the GUI renders the tree graphically using the phylotree.js library.: [evidence span withheld — anchor status: not_found]
- [results] we could reconstruct the correct phylogenetic tree in compareMS2 and MEGA11: "we could reconstruct the correct phylogenetic tree in compareMS2 and MEGA11"
- [results] The compareMS2 2.0 GUI (Figure 2) displays a phylogenetic tree with a quality metric: "The compareMS2 2.0 GUI (Figure 2) displays a phylogenetic tree with a quality metric"
