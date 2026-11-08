---
name: javascript-implementation-of-bioinformatics-algorithms
description: Use when a pairwise distance matrix (e.g., compareMS2 distances between
  LC-MS/MS datasets) must be converted into a rooted phylogenetic tree in Newick format,
  or when porting a clustering/phylogenetics algorithm into a JavaScript/Electron
  GUI.
when_to_use_negative:
- The input data already come with reference sequence or spectral-library assignments
  to specific peptides — compareMS2's direct-comparison approach targets the case
  where no (translated) genome sequence or prior spectra are available
- The goal is a non-rooted or bootstrap-supported tree from sequence alignments (use
  a dedicated tool such as MEGA11 rather than reimplementing UPGMA)
- Only a single dataset needs inspecting (e.g., internal quality via the heatmap or
  spectra2species modes) — no distance matrix or tree conversion is required
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0069
tools:
- name: compareMS2 2.0
  role: Electron/JavaScript application whose GUI computes the distance matrix from
    tandem MS datasets and draws a UPGMA tree in Newick format at the completion of
    each matrix row
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: Independent validation tool used to reconstruct the correct phylogenetic tree
    for comparison against the compareMS2 UPGMA output
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:cb46a91568405f4065ac45889ecd4d54a0f437a9d16c3389a651a755a38f436e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - MEGA11
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0069
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# javascript-implementation-of-bioinformatics-algorithms

## Summary

Implementing a classical bioinformatics algorithm (UPGMA hierarchical clustering) in JavaScript within an Electron-based tool, converting a pairwise distance matrix between tandem MS datasets into a rooted Newick tree and rendering it with phylotree.js.

## When to use

Use when a pairwise distance matrix (e.g., compareMS2 distances between LC-MS/MS datasets) must be converted into a rooted phylogenetic tree in Newick format, or when porting a clustering/phylogenetics algorithm into a JavaScript/Electron GUI.

## When NOT to use

- The input data already come with reference sequence or spectral-library assignments to specific peptides — compareMS2's direct-comparison approach targets the case where no (translated) genome sequence or prior spectra are available
- The goal is a non-rooted or bootstrap-supported tree from sequence alignments (use a dedicated tool such as MEGA11 rather than reimplementing UPGMA)
- Only a single dataset needs inspecting (e.g., internal quality via the heatmap or spectra2species modes) — no distance matrix or tree conversion is required

## Inputs

- Lower-triangular pairwise distance matrix between tandem MS (MGF) datasets, as computed by compareMS2
- Optional 'Sample to species file' (spectra_to_species.txt) mapping filenames to species names (TAB-separated)

## Outputs

- Rooted phylogenetic tree in Newick format with branch lengths after colons and a terminating semicolon
- Graphical tree rendering in the compareMS2 GUI (via phylotree.js) with a quality metric

## How to apply

Load the lower-triangular pairwise distance matrix produced by the compareMS2 pairwise spectral-comparison phase into the JavaScript codebase, initializing each tandem MS dataset as a singleton leaf cluster at height 0. Iteratively merge the two clusters with the smallest pairwise distance, recomputing the merged cluster's distance to every remaining cluster as the size-weighted average of cross-cluster member distances (standard UPGMA, not single or complete linkage). Set each merge node's height to half of the merged pair distance and derive branch lengths as parent height minus child height; serialize the rooted hierarchy in Newick format with branch lengths after colons and a terminating semicolon. In the GUI, redraw the tree with phylotree.js each time a row of the lower-triangular distance matrix completes, giving real-time feedback; validate by checking the Newick string is syntactically valid and that the topology of the five serum datasets matches the tree also reconstructed in MEGA11.

## Related tools

- **compareMS2 2.0** (Electron/JavaScript application whose GUI computes the distance matrix from tandem MS datasets and draws a UPGMA tree in Newick format at the completion of each matrix row) — https://github.com/524D/compareMS2.git
- **MEGA11** (Independent validation tool used to reconstruct the correct phylogenetic tree for comparison against the compareMS2 UPGMA output)

## Examples

```
git clone https://github.com/524D/compareMS2 && cd compareMS2 && yarn && yarn start
```

## Evaluation signals

- The emitted Newick string is syntactically valid: balanced parentheses, branch lengths after colons, and a terminating semicolon
- Applied to the five serum datasets (42,629–47,626 tandem mass spectra each), the UPGMA topology matches the correct phylogenetic tree also reconstructed in MEGA11
- Branch lengths are consistent with UPGMA node heights (each branch length equals parent height minus child height)
- The GUI redraws the UPGMA tree after completion of each row of the lower-triangular distance matrix, confirming incremental correctness of the clustering
- Distance matrix is symmetric: D(A, B) equals D(B, A), so only one direction needs computing

## Limitations

- UPGMA assumes ultrametric, clock-like data; the paper does not evaluate the matrix-to-Newick conversion in isolation, so validation relies on syntax checks and topology agreement with MEGA11
- Scaling is bounded by the pairwise-comparison phase — comparing 100 LC-MS/MS datasets requires 4950 pairwise comparisons taking several hours before any tree can be drawn
- The Jaccard-based distance is very sensitive to the precise definition of when two spectra are considered identical, which propagates into the tree topology

## Evidence

- [readme] The compareMS2 GUI provides real-time feedback by continuously updating the distance matrix, and drawing a UPGMA tree at the completion of each row in the (lower triangular) distance matrix.: [evidence span withheld — anchor status: external_doc]
- [results] The compareMS2 2.0 GUI displays a phylogenetic tree with a quality metric.: "The compareMS2 2.0 GUI (Figure 2) displays a phylogenetic tree with a quality metric"
- [results] Correct phylogenetic tree reconstruction was verified in both compareMS2 and MEGA11.: "we could reconstruct the correct phylogenetic tree in compareMS2 and MEGA11"
- [intro] The updated compareMS2 includes a GUI controlling all steps, with dynamic phylogenetic tree display and a fully symmetric distance metric.: "a graphical user interface (GUI) controlling all steps of the"
