---
name: tandem-spectral-similarity-scoring
description: Use when you have MGF peak lists from LC-MS/MS runs and need to compare
  datasets pairwise by shared/similar tandem mass spectra, estimate mass accuracy,
  charge states, isotope errors, or set m/z and similarity thresholds before any database
  search.
when_to_use_negative:
- Input spectra should be matched to predicted spectra of specific peptides for protein
  identification by database search — compareMS2 compares sets of tandem mass spectra
  to each other, not to sequence-derived spectra
- Existing spectral libraries assigned to specific peptide sequences are available
  and a library-search workflow (BiblioSpec, SpectraST) is the intended approach
- Only a single spectrum without its precursor m/z is available, so no precursor m/z
  window or m/z-difference binning can be applied
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: compareMS2
  role: Calculates spectral similarity between tandem mass spectra (eq 1) and dataset-level
    distances; provides GUI modes for phylogenetic tree, spectra2species, and heatmap
    visualizations
  repo: https://github.com/524D/compareMS2.git
- name: DISMS2
  role: Alternative direct-comparison library for tandem mass spectral similarity
    (cited as related work, not used in this workflow)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:c16216645f16155a0809807ec01aa572219555f015f822ba0aa2231675659c9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - DISMS2
  - compareMS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tandem-spectral-similarity-scoring

## Summary

Directly compares tandem mass spectra between LC-MS/MS datasets using the cosine-style spectral similarity of compareMS2 (eq 1), without database search or spectral libraries, to quantify dataset similarity, precursor m/z-dependent agreement, and data quality.

## When to use

Use when you have MGF peak lists from LC-MS/MS runs and need to compare datasets pairwise by shared/similar tandem mass spectra, estimate mass accuracy, charge states, isotope errors, or set m/z and similarity thresholds before any database search.

## When NOT to use

- Input spectra should be matched to predicted spectra of specific peptides for protein identification by database search — compareMS2 compares sets of tandem mass spectra to each other, not to sequence-derived spectra
- Existing spectral libraries assigned to specific peptide sequences are available and a library-search workflow (BiblioSpec, SpectraST) is the intended approach
- Only a single spectrum without its precursor m/z is available, so no precursor m/z window or m/z-difference binning can be applied

## Inputs

- MGF peak list files with tandem mass spectra and precursor m/z (e.g. serum datasets from ProteomeXchange PXD034932)
- Two or more LC-MS/MS datasets to compare pairwise
- R scripting examples from osf.io/jey28 for precursor m/z-difference analysis

## Outputs

- Pairwise spectral similarity scores / symmetric distance matrix between datasets
- UPGMA phylogenetic tree, spectra2species bar chart, or heatmap of spectrum inner products
- 2D histogram of precursor m/z difference versus spectral similarity (mzdiff_similarity_histogram.png)

## How to apply

Parse MGF peak list files so each tandem mass spectrum is read together with its precursor m/z, then score spectrum pairs using the spectral similarity (the cosine-style inner-product overlap in eq 1) as implemented in compareMS2 and its osf.io/jey28 R scripting examples. Restrict comparisons to spectrum pairs within a precursor m/z window, since the Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical. For dataset-level results, feed the pairwise similarities into compareMS2's symmetric distance metric (similarity 1 / distance 0 for identical spectral content) to compute a distance matrix for UPGMA phylogenetic trees, spectra2species bar charts, or heatmaps. For parameter tuning, bin spectrum pairs by precursor m/z difference (x-axis) and spectral similarity (y-axis) into a 2D histogram to estimate suitable compareMS2 m/z windows and spectral similarity thresholds. Validate by reproducing published patterns, e.g. high similarity for human vs. chimpanzee sera and low similarity for human vs. E. coli, which share few tryptic peptides.

## Related tools

- **compareMS2** (Calculates spectral similarity between tandem mass spectra (eq 1) and dataset-level distances; provides GUI modes for phylogenetic tree, spectra2species, and heatmap visualizations) — https://github.com/524D/compareMS2.git
- **DISMS2** (Alternative direct-comparison library for tandem mass spectral similarity (cited as related work, not used in this workflow))

## Evaluation signals

- Datasets with identical spectral content yield similarity 1 and distance 0; datasets with no similar spectra tend to similarity 0
- The distance metric is symmetric: D(A, B) equals D(B, A), and D(A, A) is always zero
- Human vs. chimpanzee serum comparison shows high spectral similarity; human vs. E. coli shows low similarity due to few shared tryptic peptides
- 2D histogram reproduces the paper pattern of similarity as a function of precursor m/z difference, revealing charge states, mass accuracy and isotope errors
- The heatmap computed on a single file gives consistent indications of charge states and noise level for QC

## Limitations

- The Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical
- Protein abundance (or coverage) affects the distance metric, not only amino acid sequence, so proteome similarity conflates sequence and expression differences
- Spectral clustering with these similarities tends to produce large globs of spectra rather than a distinct cluster for each peptide
- Comparing large numbers of datasets is expensive: 100 LC-MS/MS datasets require 4950 pairwise comparisons taking several hours

## Evidence

- [results] Dataset-level similarity via eq 1: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [results] Precursor m/z-dependent similarity comparison: "we compared the similarity between tandem mass spectra as a function of precursor m/z difference"
- [results] Parameter estimation from m/z-difference histogram: "used to estimate suitable parameters for compareMS2, e.g., m/z windows and spectral similarity thresholds"
- [readme] compareMS2 purpose: "compareMS2 calculates the global similarity between tandem mass spectrometry datasets."
- [readme] Heatmap mode reveals spectral properties: [evidence span withheld — anchor status: external_doc]
- [discussion] compareMS2 scope statement: "compares sets of tandem mass spectra to each other rather
than to predicted spectra of specific peptides as when identifying
proteins from tandem mass spectra."
