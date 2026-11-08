---
name: phylogenetic-tree-reconstruction-from-spectral-distances
description: Use when a lower-triangular pairwise distance matrix over tandem mass
  spectrometry datasets (e.g., MGF files) is available and a rooted tree in Newick
  format is needed for molecular phylogenetics or sample relatedness.
when_to_use_negative:
- The task requires identifying peptides or searching databases/spectral libraries
  (e.g., Mascot, SpectraST) — compareMS2 compares sets of tandem mass spectra directly
  without sequence or library assignments
- The pairwise distance matrix has not yet been computed — first run the compareMS2
  pairwise comparison phase on the MGF datasets
- The goal is spectra2species bar-chart similarity of one file against a set, or a
  self-comparison heatmap, rather than a tree over all datasets
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: Implements the UPGMA conversion of the distance matrix to Newick in JavaScript
    and renders the tree via phylotree.js in its GUI
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: Independent tool used to verify that the correct phylogenetic tree can be
    reconstructed from the same distances
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:42c066c887e4d13e87009a49b2e0e052cbc70e9d423ce126bf27680704daee3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - MEGA11
  - compareMS2
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# phylogenetic-tree-reconstruction-from-spectral-distances

## Summary

Reconstruct a rooted UPGMA phylogenetic tree from a pairwise distance matrix of tandem MS (LC-MS/MS) datasets and serialize it in Newick format, as implemented in compareMS2 2.0 (JavaScript, rendered via phylotree.js).

## When to use

Use when a lower-triangular pairwise distance matrix over tandem mass spectrometry datasets (e.g., MGF files) is available and a rooted tree in Newick format is needed for molecular phylogenetics or sample relatedness.

## When NOT to use

- The task requires identifying peptides or searching databases/spectral libraries (e.g., Mascot, SpectraST) — compareMS2 compares sets of tandem mass spectra directly without sequence or library assignments
- The pairwise distance matrix has not yet been computed — first run the compareMS2 pairwise comparison phase on the MGF datasets
- The goal is spectra2species bar-chart similarity of one file against a set, or a self-comparison heatmap, rather than a tree over all datasets

## Inputs

- Lower-triangular pairwise distance matrix between tandem MS datasets (output of compareMS2 pairwise comparisons)
- Tandem mass spectrometry dataset files (e.g., MGF) identifying the leaves/sample names
- Optional 'Sample to species file' (spectra_to_species.txt, TAB-separated filename and species name per line) to label leaves and average replicates per species

## Outputs

- Newick-format rooted tree string with branch lengths after colons and a terminating semicolon
- Graphically rendered UPGMA tree in the compareMS2 GUI (via phylotree.js) with a quality metric

## How to apply

Initialize each tandem MS dataset (e.g., MGF file) as a singleton leaf cluster with node height 0, then iteratively merge the two clusters with the smallest pairwise distance, computing each new cluster-to-cluster distance as the size-weighted average of cross-cluster member distances (standard UPGMA). Set each merge node's height to half the merged pair distance and derive branch lengths as parent height minus child height. Serialize the rooted hierarchy as a Newick string with branch lengths after colons and a terminating semicolon. In compareMS2 2.0 this is done in JavaScript, and the GUI redraws the UPGMA tree at completion of each row of the lower-triangular distance matrix, displaying a quality metric. Validation should leverage the symmetric default metric D (D(A,B) = D(B,A)), so mirrored distances need not be recomputed.

## Related tools

- **compareMS2** (Implements the UPGMA conversion of the distance matrix to Newick in JavaScript and renders the tree via phylotree.js in its GUI) — https://github.com/524D/compareMS2.git
- **MEGA11** (Independent tool used to verify that the correct phylogenetic tree can be reconstructed from the same distances)

## Examples

```
git clone https://github.com/524D/compareMS2 && cd compareMS2 && corepack enable && yarn && yarn start  # then select "phylogenetic tree" mode and press start
```

## Evaluation signals

- Emitted Newick string is syntactically valid: balanced parentheses and a terminal semicolon
- Applied to the five serum datasets (42,629–47,626 tandem mass spectra each), the topology matches the correct phylogenetic tree reported as reconstructible in both compareMS2 and MEGA11
- Each internal node's height equals half the distance between the two merged clusters, and each branch length equals parent height minus child height
- Each merge picks the two clusters with the smallest size-weighted average (UPGMA) cross-cluster distance
- The GUI progressively updates the tree after each completed row of the lower-triangular distance matrix, and the displayed quality metric is consistent with the final tree

## Limitations

- The paper does not evaluate the matrix-to-Newick conversion in isolation, so correctness must be checked via Newick syntax and topology agreement (e.g., against MEGA11)
- The default distance metric D is symmetric, so only D(A,B) for A<B is computed; inputs must not assume a full asymmetric matrix
- Tree distances reflect both peptide sequence composition and protein abundance/coverage, not sequence alone
- Building/running requires a specific NodeJS version (22.22.3); newer versions cannot currently be used due to an electron-forge bug

## Evidence

- [readme] The compareMS2 GUI provides real-time feedback by continuously updating the distance matrix, and drawing a UPGMA tree at the completion of each row in the (lower triangular) distance matrix.: [evidence span withheld — anchor status: external_doc]
- [readme] Phylogenetic tree: A set of MS2 data files are compared, and the distances between them are combined into a phylogenetic tree.: [evidence span withheld — anchor status: external_doc]
- [results] we could reconstruct the correct phylogenetic tree in compareMS2 and MEGA11: "we could reconstruct the correct phylogenetic tree in compareMS2 and MEGA11"
- [other] compareMS2 2.0 converts the distance matrix into Newick format using the UPGMA method, which is implemented in JavaScript, while the GUI renders the tree graphically using the phylotree.js library.: [evidence span withheld — anchor status: not_found]
