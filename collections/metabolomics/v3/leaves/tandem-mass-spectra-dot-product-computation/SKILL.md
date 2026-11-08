---
name: tandem-mass-spectra-dot-product-computation
description: Use when pairwise distances between proteomics samples must be quantified
  directly from peak-listed tandem mass spectra (MGF) without peptide identification
  or spectral libraries, e.g. to build a distance matrix for molecular phylogenetics
  or data quality control.
when_to_use_negative:
- Input spectra must be identified against a sequence database or spectral library
  — compareMS2 deliberately does not use existing sequence data or spectra assigned
  to specific peptide sequences (that is the Mascot/Comet or SpectraST/BiblioSpec
  class of tools)
- The goal is de novo peptide sequencing from spectra with no genomic or spectral
  prior — a LUTEFISK/PepNovo-style tool is the right class instead
- 'Samples contain no similar spectra at all: the distance tends to +infinity as set
  size grows, so the metric is uninformative for completely unrelated datasets'
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: compareMS2
  role: Directly compares all pairs of unannotated tandem mass spectra across LC-MS/MS
    datasets, computes the dot-product distributions, and converts them into a symmetric
    distance matrix (and optionally a UPGMA tree or heatmap).
  repo: https://github.com/524D/compareMS2
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:fd7943d229399e06c31a484619136c4c3abf4003865b66ab25b0f06c426daf55
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2
  tools:
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# tandem-mass-spectra-dot-product-computation

## Summary

Computes the distribution of dot products (cosine-type similarities) between all pairs of unannotated tandem mass spectra from two LC-MS/MS datasets, as in compareMS2, then summarizes it as a shared-peptide fraction that is inverted into a pairwise distance metric.

## When to use

Use when pairwise distances between proteomics samples must be quantified directly from peak-listed tandem mass spectra (MGF) without peptide identification or spectral libraries, e.g. to build a distance matrix for molecular phylogenetics or data quality control.

## When NOT to use

- Input spectra must be identified against a sequence database or spectral library — compareMS2 deliberately does not use existing sequence data or spectra assigned to specific peptide sequences (that is the Mascot/Comet or SpectraST/BiblioSpec class of tools)
- The goal is de novo peptide sequencing from spectra with no genomic or spectral prior — a LUTEFISK/PepNovo-style tool is the right class instead
- Samples contain no similar spectra at all: the distance tends to +infinity as set size grows, so the metric is uninformative for completely unrelated datasets

## Inputs

- Peak-listed tandem mass spectra files (MGF) per sample, e.g. from tryptic digests of sera analyzed by LC-MS/MS
- Optional 'spectra_to_species.txt' mapping file (TAB-separated: filename, species name) to average multiple individuals per species
- MS2 data file(s) for heatmap or spectra2species comparison modes

## Outputs

- Distribution of spectra dot products for each pair of datasets
- Per-sample-pair shared-peptide fractions (shared_peptide_fractions.csv)
- Symmetric sample-by-sample distance matrix (distance_matrix.csv)
- Species-level averaged symmetric distance matrix and phylogenetic tree (primate_tree.nwk / UPGMA display)

## How to apply

Load the peak-listed tandem mass spectra (MGF) files for each sample and, for each pair of datasets, compute the dot product between every spectrum from dataset A and every spectrum from dataset B, using all acquired spectra rather than a small annotated subset. Collect the resulting dot products into a distribution per sample pair and summarize it as the fraction of shared peptides between the two samples; invert this similarity into the default symmetric distance metric D, where identical spectral content gives similarity 1 and distance 0, and D(A,B) need not be recomputed as D(B,A). Optionally supply a 'spectra_to_species.txt' mapping (TAB-separated filename and species name per line) so that multiple sample files per species are averaged into species-level comparisons, as done for four great apes and two other primates. Assemble the lower-triangular distance matrix — the compareMS2 GUI updates it in real time and can draw a UPGMA tree per completed row — and serialize it (e.g. distance_matrix.csv or a Newick tree).

## Related tools

- **compareMS2** (Directly compares all pairs of unannotated tandem mass spectra across LC-MS/MS datasets, computes the dot-product distributions, and converts them into a symmetric distance matrix (and optionally a UPGMA tree or heatmap).) — https://github.com/524D/compareMS2

## Examples

```
git clone https://github.com/524D/compareMS2 && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- The output distance matrix must be complete and symmetric, covering every sample pair, with D(A,A) = 0 by construction
- Identical samples (same spectral content) must yield similarity 1 and distance 0
- Distances must be consistent with the computed shared-peptide fractions (distance is the inverse of similarity)
- Shared-peptide fraction values must lie within the range 0 to 1 (value_in_range check)
- Averaged species-level distances should reproduce the unique correct phylogenetic tree for the great apes and other primates when neighbor-joining/UPGMA is applied

## Limitations

- The distance metric is affected not only by peptide amino acid sequences but also by protein abundance (or coverage)
- Similarity tends to 0 (distance +infinity) as datasets with no similar spectra grow in size, so extremes are not meaningful
- Installation constraints: requires NodeJS version 22.22.3 (a later NodeJS version cannot be used due to an electron/forge bug) and corepack/yarn; Windows corepack enable needs an administrator command window
- The tool is not limited to peptide spectra, but it has seen most application to peptide tandem mass spectra

## Evidence

- [readme] defining similarity as a function of shared (similar) spectra and distance as the inverse of this similarity. Data with identical spectral content thus have similarity 1 and distance 0.: [evidence span withheld — anchor status: external_doc]
- [readme] The compareMS2 GUI provides real-time feedback by continuously updating the distance matrix, and drawing a UPGMA tree at the completion of each row in the (lower triangular) distance matrix.: [evidence span withheld — anchor status: external_doc]
- [readme] If the distance D(A, B) has already been calculated, there is no need to calculate D(B, A).: [evidence span withheld — anchor status: external_doc]
