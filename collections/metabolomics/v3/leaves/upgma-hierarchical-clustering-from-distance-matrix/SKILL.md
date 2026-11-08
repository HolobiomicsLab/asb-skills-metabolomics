---
name: upgma-hierarchical-clustering-from-distance-matrix
description: Use when you already have a pairwise distance matrix over datasets/samples
  and need a rooted tree in Newick format, as compareMS2 2.0 does with its LC-MS/MS
  dataset distances.
when_to_use_negative:
- The analysis goal is an unrooted tree or a method other than UPGMA (e.g., neighbor-joining
  or maximum likelihood), since UPGMA forces a rooted, ultrametric topology
- You only have raw LC-MS/MS data and no distance matrix — run the compareMS2 pairwise-distance
  computation first; this skill starts from the matrix
- The input is a similarity matrix rather than a distance matrix (compareMS2 defines
  distance as the inverse of spectral-content similarity), unless you invert it first
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3520
tools:
- name: compareMS2 2.0
  role: Produces the pairwise distance matrix from tandem MS datasets and converts
    it to a UPGMA tree in Newick format using JavaScript code
  repo: https://github.com/524D/compareMS2.git
- name: phylotree.js
  role: Library used by the compareMS2 GUI to render the UPGMA tree graphically
- name: MEGA11
  role: 'External validation: used to independently reconstruct the correct phylogenetic
    tree for cross-checking the compareMS2 topology'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:954217ea20c06b82cef36a6a56fcd5346e35577a4da1fd8289da7b810b68376b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - MEGA11
  - compareMS2 2.0
  - phylotree.js
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# upgma-hierarchical-clustering-from-distance-matrix

## Summary

Converts a symmetric pairwise distance matrix (e.g., the tandem MS dataset distances computed by compareMS2 2.0) into a rooted phylogenetic tree via UPGMA hierarchical clustering, serialized as a Newick string with branch lengths.

## When to use

Use when you already have a pairwise distance matrix over datasets/samples and need a rooted tree in Newick format, as compareMS2 2.0 does with its LC-MS/MS dataset distances.

## When NOT to use

- The analysis goal is an unrooted tree or a method other than UPGMA (e.g., neighbor-joining or maximum likelihood), since UPGMA forces a rooted, ultrametric topology
- You only have raw LC-MS/MS data and no distance matrix — run the compareMS2 pairwise-distance computation first; this skill starts from the matrix
- The input is a similarity matrix rather than a distance matrix (compareMS2 defines distance as the inverse of spectral-content similarity), unless you invert it first

## Inputs

- Symmetric pairwise distance matrix over tandem MS datasets (lower-triangular form from compareMS2 pairwise comparisons)
- Optional sample-to-species mapping file (TAB-separated filename/species pairs, e.g., spectra_to_species.txt) so leaves display species names and per-species replicates are averaged

## Outputs

- Rooted UPGMA phylogenetic tree in Newick format with branch lengths after colons and a terminating semicolon
- Graphical tree rendering (via phylotree.js in the compareMS2 GUI)

## How to apply

Initialize each tandem MS dataset (e.g., each MGF file of ~42,629–47,626 spectra in the serum test set) as a singleton leaf cluster at height 0. Iteratively merge the two clusters with the smallest pairwise distance, computing the new cluster's distance to every remaining cluster as the size-weighted average of cross-cluster member distances (standard UPGMA averaging, not single- or complete-linkage). Assign each merge node a height of half the merged pair distance and derive branch lengths as parent height minus child height. Serialize the rooted hierarchy as a Newick string with branch lengths after colons and a terminating semicolon; compareMS2 2.0 implements this in JavaScript and renders the tree with phylotree.js. Note the compareMS2 GUI draws the UPGMA tree incrementally, at the completion of each row of the lower-triangular distance matrix, so a partial tree is available before all comparisons finish.

## Related tools

- **compareMS2 2.0** (Produces the pairwise distance matrix from tandem MS datasets and converts it to a UPGMA tree in Newick format using JavaScript code) — https://github.com/524D/compareMS2.git
- **phylotree.js** (Library used by the compareMS2 GUI to render the UPGMA tree graphically)
- **MEGA11** (External validation: used to independently reconstruct the correct phylogenetic tree for cross-checking the compareMS2 topology)

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- Emitted Newick string is syntactically valid: balanced parentheses, branch lengths after colons, and a single terminating semicolon
- Applied to the five serum datasets, the topology matches the correct phylogenetic tree the paper reports could be reconstructed in both compareMS2 and MEGA11
- Resulting tree is ultrametric/rooted, consistent with UPGMA node heights of half the merged pair distance and branch lengths equal to parent height minus child height
- The GUI continuously redraws the tree at completion of each row of the lower-triangular distance matrix, so intermediate trees should be consistent with the rows computed so far

## Limitations

- The paper does not evaluate the matrix-to-Newick conversion in isolation, so validation rests on topology agreement with MEGA11 and Newick syntax checks
- UPGMA assumes the distance matrix is ultrametric-like; datasets whose distances violate this will yield a distorted rooted topology
- The compareMS2 distance metric depends on protein abundance/coverage as well as peptide sequence, so the tree reflects spectral-content similarity rather than pure sequence phylogeny

## Evidence

- [other] compareMS2 2.0 converts the distance matrix into Newick format using the UPGMA method, which is implemented in JavaScript: [evidence span withheld — anchor status: not_found]
- [readme] The compareMS2 GUI provides real-time feedback by continuously updating the distance matrix, and drawing a UPGMA tree at the completion of each row in the (lower triangular) distance matrix.: [evidence span withheld — anchor status: external_doc]
- [results] we could reconstruct the correct phylogenetic tree in compareMS2 and MEGA11: "we could reconstruct the correct phylogenetic tree in compareMS2 and MEGA11"
- [results] The compareMS2 GUI displays a phylogenetic tree with a quality metric: [evidence span withheld — anchor status: not_found]
