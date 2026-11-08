---
name: spectral-distance-matrix-averaging-across-individuals
description: Use when multiple MS/MS datasets (MGF tandem mass spectra files, e.g.
  sera tryptic digests) are available per taxon and a species- or group-level phylogenetic
  tree must be built from pairwise spectral comparisons.
when_to_use_negative:
- Only one dataset per taxon is available, so there are no replicate individuals to
  average over and intra-species spectral/abundance variation cannot be suppressed.
- The biological question is protein-level abundance or identification (e.g. biomarker
  discovery), since compareMS2's distance metric is influenced not only by peptide
  sequences but also by protein abundance (coverage), making it unsuitable as a quantitation
  method.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: Computes global similarity between tandem mass spectrometry datasets (spectra
    dot products) and averages results over all sample files of the same species to
    build the distance matrix used for tree reconstruction.
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:0cd3275798780880e60ebee9ee48a7813c7ee499a95a9cb71c8e89a716cd1ea8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spectral-distance-matrix-averaging-across-individuals

## Summary

Averages compareMS2 spectral dot-product-derived pairwise distances over multiple individuals (four per species in the paper) of tryptic digests of sera to yield a species-level symmetric distance matrix, from which the unique correct phylogenetic tree of great apes and other primates can be reconstructed.

## When to use

Use when multiple MS/MS datasets (MGF tandem mass spectra files, e.g. sera tryptic digests) are available per taxon and a species- or group-level phylogenetic tree must be built from pairwise spectral comparisons.

## When NOT to use

- Only one dataset per taxon is available, so there are no replicate individuals to average over and intra-species spectral/abundance variation cannot be suppressed.
- The biological question is protein-level abundance or identification (e.g. biomarker discovery), since compareMS2's distance metric is influenced not only by peptide sequences but also by protein abundance (coverage), making it unsuitable as a quantitation method.

## Inputs

- compareMS2 pairwise comparison output of spectral dot-product-derived distance values for tandem mass spectra (MGF) from tryptic digests of sera, four individuals per species
- TAB-separated sample-to-species mapping file (spectra_to_species.txt) mapping each MGF filename to a species name

## Outputs

- Averaged species-level symmetric distance matrix
- Reconstructed phylogenetic tree topology in Newick format (primate_tree.nwk)

## How to apply

Run compareMS2 (524D/compareMS2) in pairwise-comparison mode on LC-MS/MS tandem mass spectra of enzymatic (tryptic) digests, using a TAB-separated 'spectra_to_species.txt' sample-to-species mapping file so that all sample files belonging to the same species are grouped. compareMS2 defines similarity as a function of shared (similar) spectra via spectra dot products, and distance as the inverse of this similarity; supply a species map so the results of all sample files for the same species are averaged in the comparisons, producing a lower-triangular species-level distance matrix. Apply a distance-based tree reconstruction method (neighbor-joining on the averaged matrix; the compareMS2 GUI draws a UPGMA tree as rows complete) to the averaged species distance matrix. Serialize the topology in Newick format (e.g. primate_tree.nwk) and validate that the topology matches the known phylogeny of the taxa (for the paper: the unique correct tree for four great apes and two other primates).

## Related tools

- **compareMS2** (Computes global similarity between tandem mass spectrometry datasets (spectra dot products) and averages results over all sample files of the same species to build the distance matrix used for tree reconstruction.) — https://github.com/524D/compareMS2.git

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- The species-level distance matrix is symmetric with a zero diagonal, since compareMS2's default distance metric D is symmetric and D(A, A) is always zero.
- The reconstructed Newick topology matches the unique correct phylogenetic tree for the great apes and other primates reported in the paper.
- Each species' distances are averages over exactly four individuals, verifiable by checking the sample-to-species mapping covers all four sera files per species.
- Datasets with identical spectral content yield similarity 1 and distance 0, providing a sanity check of the metric.
- The primate sera test datasets (OSF sg796, 1,000 tandem mass spectra per sample; PRIDE PXD034932) reproduce the expected tree.

## Limitations

- The distance metric depends not only on amino acid sequences of peptides but also on the abundance (or coverage) of proteins, so tissue/cell-line differences can confound phylogenetic interpretation.
- The similarity of datasets with no similar spectra tends to 0 (distance +infinity) as set size grows, so very dissistant pairs can yield unstable distances.
- Requires multiple individuals per species (four per species in the paper) for the averaging step to recover the correct topology.

## Evidence

- [readme] sample-to-species averaging mechanism: [evidence span withheld — anchor status: not_found]
- [readme] distance matrix tree feedback: [evidence span withheld — anchor status: external_doc]
- [readme] similarity-distance definition: [evidence span withheld — anchor status: external_doc]
