---
name: peak-list-spectra-parsing
description: Use when LC-MS/MS tandem mass spectra are provided as peak-list files
  (e.g., MGF) that must be loaded and parsed as unannotated spectra for direct pairwise
  comparison, dot-product distributions, or distance-matrix computation in compareMS2.
when_to_use_negative:
- Input is raw LC-MS/MS instrument data rather than peak-listed spectra (MGF); the
  spectra must first be converted to peak lists.
- The goal is peptide sequence identification via database search (e.g., Mascot, Comet)
  or spectral libraries (e.g., SpectraST); compareMS2 deliberately uses no sequence
  data or spectra assigned to a specific peptide sequence.
- Input is already a computed distance matrix or an annotated/identified peptide table
  rather than unannotated tandem spectra.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: compareMS2
  role: Loads peak-listed (MGF) tandem mass spectra and computes dot products between
    all spectrum pairs across datasets to yield the pairwise distance matrix
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:48aef9476e47a393d17f7d9bf40bb2d53ebfeeb45f22e3aca91691cf32f10124
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# peak-list-spectra-parsing

## Summary

Loads and parses peak-listed (MGF) tandem MS/MS spectra as unannotated input for the compareMS2 direct-comparison workflow, in which dot products between all spectrum pairs from two samples quantify shared peptides and are inverted into pairwise distances.

## When to use

Use when LC-MS/MS tandem mass spectra are provided as peak-list files (e.g., MGF) that must be loaded and parsed as unannotated spectra for direct pairwise comparison, dot-product distributions, or distance-matrix computation in compareMS2.

## When NOT to use

- Input is raw LC-MS/MS instrument data rather than peak-listed spectra (MGF); the spectra must first be converted to peak lists.
- The goal is peptide sequence identification via database search (e.g., Mascot, Comet) or spectral libraries (e.g., SpectraST); compareMS2 deliberately uses no sequence data or spectra assigned to a specific peptide sequence.
- Input is already a computed distance matrix or an annotated/identified peptide table rather than unannotated tandem spectra.

## Inputs

- Peak-listed tandem mass spectra files (e.g., MGF) from LC-MS/MS runs, one or more per sample dataset
- Optional sample-to-species mapping file 'spectra_to_species.txt' (one TAB-separated filename and species name per line)

## Outputs

- Parsed unannotated tandem mass spectra (peak m/z and intensity lists) ready for pairwise dot-product comparison
- Per-sample-pair dot-product distributions summarized as shared-peptide fractions and inverted into a symmetric sample-by-sample distance matrix

## How to apply

Load the peak-listed tandem mass spectra files (MGF) for each sample dataset, keeping every acquired MS2 spectrum with its peak m/z and intensity list and without peptide annotation, database search, or spectral-library lookup, since compareMS2 compares tandem mass spectra directly with other tandem mass spectra. For each pair of samples, compute the dot product (cosine-type inner product) between every spectrum pair, one taken from each dataset, using all acquired data rather than a small annotated subset. Collect the dot products into a per-sample-pair distribution, summarize it as the fraction of shared peptides, and invert this similarity to distance (identical spectral content gives similarity 1 and distance 0). If a sample-to-species file named spectra_to_species.txt is present (one filename and species name per line, TAB-separated), files of the same species are averaged in the comparisons. Exploit the metric's symmetry, D(A,B) = D(B,A) and D(A,A) = 0, to compute only the lower-triangular distance matrix, which compareMS2 then feeds to UPGMA tree drawing.

## Related tools

- **compareMS2** (Loads peak-listed (MGF) tandem mass spectra and computes dot products between all spectrum pairs across datasets to yield the pairwise distance matrix) — https://github.com/524D/compareMS2.git

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; yarn; yarn start
```

## Evaluation signals

- Every acquired MS2 spectrum in the MGF peak lists is parsed, with no filtering to annotated or database-identified peptides (unbiased use of all acquired data).
- Parsed spectra retain peak m/z and intensity lists suitable for dot-product (cosine-type) computation between dataset pairs.
- Resulting distance matrix is symmetric with zero diagonal: D(A,B) = D(B,A) and D(A,A) = 0; datasets with identical spectral content yield similarity 1 and distance 0.
- Each sample pair's distance equals the inversion of the shared-peptide fraction summarized from its dot-product distribution, and every sample pair appears in the matrix.
- When spectra_to_species.txt is supplied, results of all sample files for the same species are averaged, as the README specifies.

## Limitations

- compareMS2 provides relatedness, not peptide identifications: it uses no existing sequence data or spectral libraries of spectra assigned to specific peptide sequences.
- The distance metric reflects both peptide amino acid sequences and protein abundance/coverage, so proteome abundance differences also move distances.
- Requires peak-listed spectra (e.g., MGF); raw instrument files must be converted first, and for datasets with no similar spectra the similarity tends to 0 (distance +infinity) as set size grows, though such extremes are unlikely in reality.

## Evidence

- [abstract] Dot-product distributions quantify shared peptides between samples: "The distribution of spectra dot products for thousands of tandem mass spectra collected from two samples provides a measure on the fraction of shared peptides between the two samples."
- [abstract] All acquired spectra are used, not a small annotated subset: "the unbiased use of all acquired data rather than a small subset of spectra, and the potential use of heavily degraded proteins or proteins with a priori unknown modifications."
- [readme] Similarity-to-distance definition and identical-content anchor from README: [evidence span withheld — anchor status: external_doc]
- [readme] MGF peak-list files are the compared input format: [evidence span withheld — anchor status: external_doc]
- [readme] Distance metric symmetry and lower-triangular computation: [evidence span withheld — anchor status: external_doc]
