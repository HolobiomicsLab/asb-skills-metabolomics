---
name: newick-tree-format-serialization
description: Use when a phylogenetic analysis has produced a species-level distance
  matrix (e.g. from averaged compareMS2 pairwise spectra comparisons) and the reconstructed
  topology must be exported as a Newick (.nwk) file for downstream inspection or validation.
when_to_use_negative:
- The input is raw spectral data and no pairwise compareMS2 distance matrix has yet
  been computed — tree serialization presupposes distances are already available
- The goal is peptide/protein identification by database search rather than distance-based
  phylogenetic comparison — compareMS2 deliberately does not use sequence databases
  or spectral libraries
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: Computes global similarity between tandem mass spectrometry datasets, averages
    results across samples of the same species via the sample-to-species file, and
    draws UPGMA phylogenetic trees whose topology is serialized to Newick
  repo: https://github.com/524D/compareMS2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:e8bcd59f4d4c9fe640fb33fa1b058d9df6570dc4e39384b0ac36f1fbeb65f67e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# newick-tree-format-serialization

## Summary

Serialize a distance-based phylogenetic tree (e.g. UPGMA or neighbor-joining over compareMS2 spectra dot-product distances, averaged across individuals per species) into the Newick text format so the reconstructed primate topology can be stored, shared, and validated against the published unique correct tree.

## When to use

Use when a phylogenetic analysis has produced a species-level distance matrix (e.g. from averaged compareMS2 pairwise spectra comparisons) and the reconstructed topology must be exported as a Newick (.nwk) file for downstream inspection or validation.

## When NOT to use

- The input is raw spectral data and no pairwise compareMS2 distance matrix has yet been computed — tree serialization presupposes distances are already available
- The goal is peptide/protein identification by database search rather than distance-based phylogenetic comparison — compareMS2 deliberately does not use sequence databases or spectral libraries

## Inputs

- compareMS2 pairwise comparison output containing spectra dot-product-derived distance values for tryptic digests of sera (MGF/LC-MS/MS dataset files)
- Sample-to-species mapping file (TAB-separated filename and species per line, e.g. spectra_to_species.txt) for averaging over four individuals per species
- Averaged species-level symmetric distance matrix

## Outputs

- Newick-format tree file (e.g. primate_tree.nwk) encoding the reconstructed phylogenetic topology with species leaf labels
- Optionally the intermediate averaged symmetric distance matrix

## How to apply

First produce the species-level symmetric distance matrix by averaging the compareMS2 spectra dot-product-derived distance values over the four individuals per species (compareMS2's 'Sample to species file', a TAB-separated sample-to-species mapping such as spectra_to_species.txt, performs this averaging automatically). Then apply a distance-based tree reconstruction method — the compareMS2 GUI draws UPGMA trees in real time, while the paper's workflow specifies neighbor-joining — to obtain the topology. Serialize the resulting topology in Newick format, e.g. writing the tree to primate_tree.nwk, with species names as leaf labels. Validate correctness by checking that the Newick topology reproduces the unique correct phylogenetic tree for the great apes and other primates reported in the paper.

## Related tools

- **compareMS2** (Computes global similarity between tandem mass spectrometry datasets, averages results across samples of the same species via the sample-to-species file, and draws UPGMA phylogenetic trees whose topology is serialized to Newick) — https://github.com/524D/compareMS2

## Evaluation signals

- The Newick file parses and its leaf set equals the six species (four great apes and two other primates) analyzed in the study
- The serialized topology matches the unique correct phylogenetic tree for the great apes and other primates reported in the paper
- The distance matrix underlying the tree is symmetric and each species' distances are the average over four individuals per species
- Identical datasets have similarity 1 and distance 0, so self-distances in the matrix are zero before tree reconstruction
- Tree reconstruction is deterministic given the averaged distance matrix, so re-running UPGMA/neighbor-joining reproduces the same Newick string

## Limitations

- The compareMS2 GUI draws UPGMA trees by default; obtaining the neighbor-joining topology described in the paper may require exporting the distance matrix and applying neighbor-joining separately
- The compareMS2 distance metric depends not only on peptide amino acid sequences but also on protein abundance (coverage), so tree topology can be affected by proteome composition differences
- Datasets with no similar spectra have distance tending to infinity as set size grows, which can destabilize tree reconstruction for very distant taxa

## Evidence

- [abstract] Pairwise comparison between species with averaging over four individuals per species enabled reconstruction of the unique correct phylogenetic tree for the great apes and other primates.: "By pairwise comparison between species and averaging over four individuals per species, it was possible to reconstruct the unique correct phylogenetic tree for the great apes and other primates."
- [other] Serialize the reconstructed topology in Newick format to primate_tree.nwk.: [evidence span withheld — anchor status: not_found]
- [readme] compareMS2 GUI provides real-time feedback by continuously updating the distance matrix, and drawing a UPGMA tree at the completion of each row in the (lower triangular) distance matrix.: [evidence span withheld — anchor status: external_doc]
- [readme] The sample-to-species file maps file names to species names and averages all sample files for the same species in the comparisons.: [evidence span withheld — anchor status: external_doc]
