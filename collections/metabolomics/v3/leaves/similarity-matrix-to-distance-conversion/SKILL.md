---
name: similarity-matrix-to-distance-conversion
description: Use when a pairwise similarity matrix from unannotated tandem mass spectra
  comparisons (e.g. compareMS2 dot products) must be inverted into distances and aggregated
  per species to build a phylogenetic tree or quantify shared peptide content.
when_to_use_negative:
- Input is already a distance matrix or phylogenetic distance table — no similarity-to-distance
  inversion is needed
- Spectra have already been assigned to peptide sequences by database search or spectral
  library search — sequence-based phylogenetics tools are more appropriate
- Datasets contain few or no similar spectra between samples, where similarity tends
  to 0 and distances become degenerate (+∞)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: Calculates the global similarity (dot products of tandem mass spectra) between
    datasets and converts similarities into distances, averaging over sample files
    of the same species via the sample-to-species file
  repo: https://github.com/524D/compareMS2.git
- name: git
  role: Used to clone the compareMS2 repository for installation
  repo: https://github.com/524D/compareMS2.git
- name: yarn
  role: Dependency installation and running the compareMS2 app in development mode
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:4ecb67f7664fb1e23be0f972be7c5e604bd20a83b6457a2b86b8b08f9be388aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  - git
  - yarn
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# similarity-matrix-to-distance-conversion

## Summary

Converts pairwise dot-product-based similarity values from compareMS2 tandem mass spectra comparisons into a distance metric suitable for phylogenetic reconstruction, mapping sample pairs to species labels and averaging over individuals per species.

## When to use

Use when a pairwise similarity matrix from unannotated tandem mass spectra comparisons (e.g. compareMS2 dot products) must be inverted into distances and aggregated per species to build a phylogenetic tree or quantify shared peptide content.

## When NOT to use

- Input is already a distance matrix or phylogenetic distance table — no similarity-to-distance inversion is needed
- Spectra have already been assigned to peptide sequences by database search or spectral library search — sequence-based phylogenetics tools are more appropriate
- Datasets contain few or no similar spectra between samples, where similarity tends to 0 and distances become degenerate (+∞)

## Inputs

- Pairwise similarity matrix of dot-product values from compareMS2 comparison of LC-MS/MS tandem mass spectra (MGF files from serum tryptic digests)
- Sample-to-species mapping file (spectra_to_species.txt), TAB-separated filename and species name per line
- Species and individual labels for each sample (four individuals per species)

## Outputs

- Per-individual-pair shared-peptide fraction table (shared_peptide_fractions.csv)
- Per-species-pair averaged shared-peptide fractions
- Symmetric lower-triangular distance matrix usable for UPGMA phylogenetic tree construction

## How to apply

Load the pairwise similarity matrix produced by the compareMS2 dot-product comparison of tandem mass spectra (e.g. from serum tryptic digests) and map each matrix entry to its sample, individual, and species labels using a sample-to-species mapping file (spectra_to_species.txt, TAB-separated filename/species pairs). Convert each dot-product-derived similarity into the fraction of shared peptides it represents; similarity 1 corresponds to identical spectral content and distance 0, while similarity trending to 0 corresponds to distance +∞, so the distance is the inverse of the similarity. Average the pairwise values over the four individuals per species to obtain species-level shared-peptide fractions — the compareMS2 GUI performs this averaging automatically when a 'Sample to species file' is supplied. Write the per-individual-pair fractions and per-species-pair averages to shared_peptide_fractions.csv, then feed the distance matrix into UPGMA tree construction, which is what allowed reconstruction of the unique correct phylogenetic tree for the great apes and other primates.

## Related tools

- **compareMS2** (Calculates the global similarity (dot products of tandem mass spectra) between datasets and converts similarities into distances, averaging over sample files of the same species via the sample-to-species file) — https://github.com/524D/compareMS2.git
- **git** (Used to clone the compareMS2 repository for installation) — https://github.com/524D/compareMS2.git
- **yarn** (Dependency installation and running the compareMS2 app in development mode)

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- Output table shared_peptide_fractions.csv exists and contains one row per sample pair
- All shared-peptide fraction values fall within the range 0 to 1 (value_in_range check)
- Distance metric is symmetric: D(A,B) equals D(B,A), and D(A,A) is zero
- Datasets with identical spectral content yield similarity 1 and distance 0
- UPGMA tree built from species-averaged distances reproduces the unique correct phylogenetic tree for the great apes and other primates

## Limitations

- The distance metric is affected not only by peptide amino acid sequences but also by protein abundance (or coverage)
- Similarity of datasets with no similar spectra tends to 0 (distance +∞) as the size of the sets goes to infinity, so extreme values are unstable
- The extremes of none or all spectra being similar between two LC-MS/MS datasets are unlikely to occur in reality

## Evidence

- [readme] Each line should contain a filename and a species name, separated by a TAB character. If the directory with samples contains a file with the exact name "spectra_to_species.txt": [evidence span withheld — anchor status: external_doc]
- [readme] In the comparisons, the results of all samples files for the same species are then averaged.: [evidence span withheld — anchor status: external_doc]
- [readme] Data with identical spectral content thus have similarity 1 and distance 0.: [evidence span withheld — anchor status: external_doc]
- [readme] not only the amino acid sequences of the peptides affect the distance metric in compareMS2, but also the abundance (or coverage) of the proteins: [evidence span withheld — anchor status: external_doc]
