---
name: shared-peptide-fraction-estimation
description: Use when you need a quantitative sample-to-sample peptide-sharing measure
  from raw tandem mass spectra (e.g. MGF peak lists) without any database search,
  peptide annotation, or spectral library, e.g. for phylogenetics or QC.
when_to_use_negative:
- Your analysis requires peptide sequences or protein identifications — compareMS2
  deliberately does not use database searches or spectral libraries and works directly
  on unannotated spectra
- You need to distinguish samples solely by protein abundance/coverage rather than
  shared peptide content — the README notes that not only sequence but also abundance
  (coverage) of proteins affects the distance metric
- Input is already annotated spectra or a feature/peptide table; this skill computes
  similarity from raw tandem mass spectra peak lists
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3292
tools:
- name: compareMS2
  role: Computes the global similarity between tandem MS datasets by comparing all
    pairs of unannotated tandem mass spectra, summarizing dot-product distributions
    into shared-peptide fractions and an inverted distance matrix
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:ac88f6594da594cd9d750fed97ca8a80bea78c064e4d4c79ec8ffa3eb7307ebc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# shared-peptide-fraction-estimation

## Summary

Estimates the fraction of shared peptides between two samples by building the distribution of dot products (inner products) between all pairs of unannotated tandem mass spectra, then inverting that similarity into a symmetric pairwise distance metric as implemented in compareMS2.

## When to use

Use when you need a quantitative sample-to-sample peptide-sharing measure from raw tandem mass spectra (e.g. MGF peak lists) without any database search, peptide annotation, or spectral library, e.g. for phylogenetics or QC.

## When NOT to use

- Your analysis requires peptide sequences or protein identifications — compareMS2 deliberately does not use database searches or spectral libraries and works directly on unannotated spectra
- You need to distinguish samples solely by protein abundance/coverage rather than shared peptide content — the README notes that not only sequence but also abundance (coverage) of proteins affects the distance metric
- Input is already annotated spectra or a feature/peptide table; this skill computes similarity from raw tandem mass spectra peak lists

## Inputs

- Peak-listed tandem mass spectrometry (LC-MS/MS) data files per sample, e.g. MGF peak lists of unannotated MS2 spectra
- Optional 'Sample to species file' (spectra_to_species.txt) mapping filenames to species names (TAB-separated, one filename and species per line)

## Outputs

- Dot-product (inner-product) distribution for each sample pair
- Fraction of shared peptides between each pair of samples
- Symmetric sample-by-sample distance matrix (lower triangular; identical samples have distance 0)
- Downstream UPGMA phylogenetic tree or heatmap visualizations

## How to apply

Load peak-listed tandem mass spectra files (MGF-style peak lists) for each sample dataset to be compared. For every pair of samples, compute the dot product (cosine-type spectral similarity) between every pair of unannotated tandem mass spectra, one taken from each dataset, using all acquired spectra rather than a small annotated subset — this unbiased use of all data also covers heavily degraded proteins or proteins with a priori unknown modifications. Summarize the resulting dot-product distribution as the fraction of shared peptides between the two samples. Invert that similarity to obtain the distance metric D (identical spectral content gives similarity 1 and distance 0; datasets with no similar spectra tend to distance +∞ as set size grows). Assemble all pairwise distances into a symmetric lower-triangular distance matrix (D(A,B) = D(B,A), D(A,A) = 0, so already-computed values need not be recomputed), which can feed a UPGMA tree; with a spectra_to_species.txt file, results from multiple sample files of the same species are averaged.

## Related tools

- **compareMS2** (Computes the global similarity between tandem MS datasets by comparing all pairs of unannotated tandem mass spectra, summarizing dot-product distributions into shared-peptide fractions and an inverted distance matrix) — https://github.com/524D/compareMS2.git

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- The distance matrix is symmetric and complete over every sample pair; D(A,A) is always zero and identical datasets yield similarity 1 and distance 0
- Identical samples produce zero distance; datasets with no similar spectra approach similarity 0 (distance +∞) as set sizes grow
- Each distance value is consistent with the shared-peptide fraction implied by the underlying dot-product distribution (distance is the inverse of similarity)
- Reproduction of the article's benchmark: pairwise comparison of great ape and other primate sera tryptic digests (four individuals per species) reconstructs the unique correct phylogenetic tree

## Limitations

- The distance metric depends on protein abundance (or coverage) as well as amino acid sequence, so samples with the same proteome but different protein expression levels will not appear identical
- Similarity tends to 0 (distance +∞) for datasets with no similar spectra only asymptotically as set sizes go to infinity; extremes of none or all spectra being similar are unlikely in real data
- Because all acquired data are used, results reflect acquisition depth; the GUI averages multiple sample files per species only when a spectra_to_species.txt file is supplied

## Evidence

- [readme] Distance definition: similarity 1/distance 0 for identical spectral content; no similar spectra tends to distance +∞: [evidence span withheld — anchor status: external_doc]
- [readme] Distance metric is symmetric; D(A,B) = D(B,A), D(A,A) always zero: [evidence span withheld — anchor status: external_doc]
