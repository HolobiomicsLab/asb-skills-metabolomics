---
name: cross-species-peptide-overlap-analysis
description: Use when you have unannotated LC-MS/MS tandem mass spectra (e.g. MGF
  files from serum tryptic digests) from multiple species or individuals and need
  to quantify shared peptide content — without database search — as a distance for
  phylogenetic reconstruction or species identification.
when_to_use_negative:
- When translated genome sequences or spectral libraries are available and an identification-based
  comparison (database search like Mascot/Comet or spectral library search like SpectraST)
  is preferred over direct spectral comparison
- When the input is not tandem mass spectra (e.g. MS1 feature tables, protein abundance
  matrices) — compareMS2 operates on MS2 spectra
- When the goal is absolute protein identification or quantification rather than unannotated
  pairwise similarity between samples
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0194
tools:
- name: compareMS2
  role: Calculates the global similarity between tandem mass spectrometry datasets
    via spectra dot products, converts similarity to distance, averages samples per
    species, and builds the phylogenetic tree
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:00bf1d1a6368f6da28c8dcd38cc07416e6f7d930a77f499461e01ce3d442f5ef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-species-peptide-overlap-analysis

## Summary

Quantify the fraction of shared peptides between biological samples by comparing unannotated tandem mass spectra directly with compareMS2, inverting the dot-product-derived similarity into a distance metric, and averaging over individuals per species to support molecular phylogenetics without any peptide identification.

## When to use

Use when you have unannotated LC-MS/MS tandem mass spectra (e.g. MGF files from serum tryptic digests) from multiple species or individuals and need to quantify shared peptide content — without database search — as a distance for phylogenetic reconstruction or species identification.

## When NOT to use

- When translated genome sequences or spectral libraries are available and an identification-based comparison (database search like Mascot/Comet or spectral library search like SpectraST) is preferred over direct spectral comparison
- When the input is not tandem mass spectra (e.g. MS1 feature tables, protein abundance matrices) — compareMS2 operates on MS2 spectra
- When the goal is absolute protein identification or quantification rather than unannotated pairwise similarity between samples

## Inputs

- MGF or other tandem mass spectrometry (LC-MS/MS) data files, e.g. tryptic digests of sera from great apes and other primates (test data: OSF sg796, PRIDE PXD034932)
- spectra_to_species.txt: plain-text sample-to-species mapping file with filename and species name per line, TAB-separated
- Pairwise dot-product similarity matrix produced by compareMS2

## Outputs

- shared_peptide_fractions.csv with one row per sample pair: per-individual-pair shared-peptide fractions and per-species-pair averages
- Symmetric lower-triangular distance matrix (D) suitable for UPGMA phylogenetic tree construction
- UPGMA phylogenetic tree (e.g. circular tree of primate sera datasets)

## How to apply

Run compareMS2 in pairwise mode over the LC-MS/MS datasets (e.g. MGF files of tryptic digests of sera from four individuals per species); it computes the global similarity between datasets via dot products of thousands of tandem mass spectra. Supply a 'Sample to species file' (spectra_to_species.txt) where each line contains a filename and species name separated by a TAB character, so that results from all sample files of the same species are averaged. Convert each dot-product similarity into the fraction of shared peptides it represents and invert it into the default symmetric distance metric D (identical spectral content gives similarity 1 and distance 0; no similar spectra drives distance toward +∞). Write the per-individual-pair fractions and per-species-pair averages to shared_peptide_fractions.csv. Rationale: comparing tandem mass spectra directly avoids dependence on genome sequences or spectral libraries, uses all acquired data (including degraded proteins or proteins with unknown modifications), and the resulting distance matrix supports UPGMA tree construction — this reproduced the unique correct phylogenetic tree for great apes and other primates.

## Related tools

- **compareMS2** (Calculates the global similarity between tandem mass spectrometry datasets via spectra dot products, converts similarity to distance, averages samples per species, and builds the phylogenetic tree) — https://github.com/524D/compareMS2.git

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- Validation check: shared_peptide_fractions.csv exists, contains one row per pair, and all shared-peptide-fraction values are within the range 0 to 1 (value_in_range check)
- Datasets with identical spectral content must yield similarity 1 and distance 0; the distance matrix must be symmetric with zero diagonal (D(A,A) = D(B,B) = 0)
- Reconstructed UPGMA tree from species-averaged distances must match the known phylogeny (unique correct tree for great apes and other primates)
- The primate sera dataset of 1,000 tandem mass spectra per sample (OSF sg796 / PRIDE PXD034932) reproduces the published tree as a positive control

## Limitations

- The distance metric reflects not only peptide amino acid sequence but also protein abundance (or coverage), so differences in proteome composition across tissues or conditions confound purely phylogenetic interpretation
- Similarity of datasets with no similar spectra tends to 0 (distance +∞) as set size goes to infinity; the extremes of none or all spectra being similar are unlikely in practice
- Installation is version-sensitive: a known Electron Forge bug requires NodeJS version 22.22.3 (the then-latest NodeJS cannot be used), and corepack enable requires administrator rights on Windows

## Evidence

- [abstract] dot-product similarity to shared-peptide fraction: "The distribution of spectra dot products for thousands of tandem mass spectra collected from two samples provides a measure on the fraction of shared peptides between the two samples."
- [abstract] phylogenetic reconstruction success: "By pairwise comparison between species and averaging over four individuals per species, it was possible to reconstruct the unique correct phylogenetic tree for the great apes and other primates."
- [readme] compareMS2 definition of similarity and distance: [evidence span withheld — anchor status: external_doc]
- [readme] sample-to-species averaging file format: [evidence span withheld — anchor status: external_doc]
- [abstract] unbiased use of all acquired data: "the unbiased use of all acquired data rather than a small subset of spectra, and the potential use of heavily degraded proteins or proteins with a priori unknown modifications."
