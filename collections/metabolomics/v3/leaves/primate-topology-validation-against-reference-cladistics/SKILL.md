---
name: primate-topology-validation-against-reference-cladistics
description: Use when LC-MS/MS tandem mass spectra (e.g. MGF files of tryptic digests)
  are available for multiple individuals per species and the goal is a species-level
  phylogeny validated against known primate cladistics.
when_to_use_negative:
- When the goal is peptide/protein identification rather than phylogenetics — compareMS2
  compares spectra directly without database or spectral library search
- When samples are single files per species without individuals to average — within-species
  spectral variance will not be averaged out, weakening the distance metric
- When the sample-to-species mapping is absent and species-level averaging over multiple
  files is required — averaging only occurs when the mapping file is supplied
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: Computes pairwise spectra dot-product similarities between tandem MS datasets
    and builds the distance matrix / UPGMA tree for phylogenetic reconstruction
  repo: https://github.com/524D/compareMS2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:07bb71b0546a4663ee7d716033a06799d8d7be2a60c73949cb85cf8749e1e5f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# primate-topology-validation-against-reference-cladistics

## Summary

Reconstruct and validate the unique correct phylogenetic tree of great apes and other primates from compareMS2 pairwise tandem mass spectra distances, averaging spectral dot-product-derived distances over four individuals per species and applying a distance-based tree method.

## When to use

Use when LC-MS/MS tandem mass spectra (e.g. MGF files of tryptic digests) are available for multiple individuals per species and the goal is a species-level phylogeny validated against known primate cladistics.

## When NOT to use

- When the goal is peptide/protein identification rather than phylogenetics — compareMS2 compares spectra directly without database or spectral library search
- When samples are single files per species without individuals to average — within-species spectral variance will not be averaged out, weakening the distance metric
- When the sample-to-species mapping is absent and species-level averaging over multiple files is required — averaging only occurs when the mapping file is supplied

## Inputs

- compareMS2 pairwise comparison output containing spectra dot-product-derived distance values
- MGF-format LC-MS/MS tandem mass spectra files from tryptic digests of sera, four individuals per species
- spectra_to_species.txt sample-to-species mapping file (filename TAB species name per line)

## Outputs

- Species-level symmetric distance matrix averaged over four individuals per species
- Phylogenetic tree in Newick format (primate_tree.nwk)
- Topology validated against the unique correct primate cladistics reference tree

## How to apply

Run compareMS2 in phylogenetic tree mode on the tryptic digests of sera from the four great apes and two other primates, supplying a spectra_to_species.txt sample-to-species mapping so pairwise spectral dot-product distances are averaged over the four individuals per species. Build the species-level symmetric distance matrix from the inverted dot-product similarity values (identical spectral content gives distance 0), letting the GUI update the matrix row by row. Apply a distance-based tree reconstruction (neighbor-joining on the averaged matrix; the compareMS2 GUI draws UPGMA as each row completes) and serialize the topology in Newick format (primate_tree.nwk). Validate by checking that the reconstructed topology matches the unique correct primate tree, i.e. the accepted sister-group relationships among human, chimpanzee, gorilla, orangutan and the two other primates.

## Related tools

- **compareMS2** (Computes pairwise spectra dot-product similarities between tandem MS datasets and builds the distance matrix / UPGMA tree for phylogenetic reconstruction) — https://github.com/524D/compareMS2

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- Reconstructed Newick topology in primate_tree.nwk matches the unique correct primate tree reported in the paper
- Distance matrix is symmetric and has zero diagonal (identical datasets give distance 0)
- Species-level distances are averaged over exactly four individuals per species via the sample-to-species file
- GUI continuously updates the distance matrix and draws the UPGMA tree at completion of each row, allowing incremental inspection of the lower-triangular matrix
- Test against the public primate sera datasets (OSF sg796 / PRIDE PXD034932) reproduces the tree in Figure 1 of the README

## Limitations

- The distance metric depends not only on peptide sequence but also on protein abundance/coverage of the compared samples
- Distances tend to infinity for datasets with no similar spectra, so unrelated species pairs are bounded only by dataset size
- compareMS2 performs direct spectral comparison without sequence or spectral-library priors, so no protein identifications are produced as a byproduct

## Evidence

- [readme] compareMS2 similarity/distance definition: [evidence span withheld — anchor status: external_doc]
- [readme] Sample-to-species averaging mechanism: [evidence span withheld — anchor status: external_doc]
- [readme] Tree drawing behavior in the GUI: [evidence span withheld — anchor status: external_doc]
