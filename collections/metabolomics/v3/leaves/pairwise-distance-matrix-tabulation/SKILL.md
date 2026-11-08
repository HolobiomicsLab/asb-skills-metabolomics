---
name: pairwise-distance-matrix-tabulation
description: Use when you have peak-listed LC-MS/MS (MGF) datasets for multiple samples
  and need a quantitative pairwise similarity/distance matrix without peptide identification
  or spectral annotation.
when_to_use_negative:
- The analysis goal requires peptide or protein identification — compareMS2 does not
  assign sequences, unlike database search (Mascot, Comet) or spectral library tools
- Input is already an identified feature/peptide abundance table rather than raw peak-listed
  tandem spectra
- Only one sample is available with no pairwise comparison target (self-comparison
  yields distance 0 trivially)
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3292
tools:
- name: compareMS2
  role: Computes global similarity between tandem mass spectrometry datasets via dot-product
    distributions of all unannotated MS2 spectra, producing the pairwise distance
    matrix
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:c6de5fc661c6ff27b55e361e6292aae21ea98781952326c2018801b874f66f9c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pairwise-distance-matrix-tabulation

## Summary

Assemble a symmetric sample-by-sample distance matrix from pairwise tandem-MS/MS spectral similarities: dot-product distributions over all acquired unannotated spectra yield a shared-peptide fraction, whose inverse is the distance, tabulated for downstream phylogenetic reconstruction or QC.

## When to use

Use when you have peak-listed LC-MS/MS (MGF) datasets for multiple samples and need a quantitative pairwise similarity/distance matrix without peptide identification or spectral annotation.

## When NOT to use

- The analysis goal requires peptide or protein identification — compareMS2 does not assign sequences, unlike database search (Mascot, Comet) or spectral library tools
- Input is already an identified feature/peptide abundance table rather than raw peak-listed tandem spectra
- Only one sample is available with no pairwise comparison target (self-comparison yields distance 0 trivially)

## Inputs

- Peak-listed tandem mass spectrometry (LC-MS/MS) dataset files per sample, e.g. MGF files
- Optional spectra_to_species.txt mapping file (TAB-separated filename and species name per line)

## Outputs

- Symmetric sample-by-sample distance matrix (distance_matrix.csv)
- Per-individual-pair and per-species-pair shared-peptide fraction table (shared_peptide_fractions.csv)

## How to apply

Load the peak-listed tandem mass spectra (e.g. MGF files from serum tryptic digests) for each sample. For every sample pair, compute the dot product (cosine-type spectral similarity) between every pair of unannotated MS2 spectra, one from each dataset, using compareMS2 and all acquired spectra rather than a small annotated subset. Summarize the resulting dot-product distribution as the fraction of shared peptides, then invert it into the distance metric (identical spectral content gives similarity 1 and distance 0). Optionally supply a 'sample to species' file (spectra_to_species.txt, tab-separated filename/species lines) to average results over individuals of the same species. Assemble all pairwise distances into a complete symmetric matrix — D(A,B)=D(B,A) and D(A,A)=0 — writing per-pair fractions and species-level averages (e.g. shared_peptide_fractions.csv) alongside distance_matrix.csv.

## Related tools

- **compareMS2** (Computes global similarity between tandem mass spectrometry datasets via dot-product distributions of all unannotated MS2 spectra, producing the pairwise distance matrix) — https://github.com/524D/compareMS2.git

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- Output matrix is complete and symmetric, covering every sample pair, with zero diagonal (identical samples yield zero distance)
- All shared-peptide fraction values fall within 0 to 1 (value_in_range check on the shared-peptide-fraction column)
- Distances are consistent with the computed shared-peptide fractions (distance = inverse of similarity)
- When a spectra_to_species mapping is used, species-level averages are averaged over the four individuals per species as in the primate sera study
- The resulting distance matrix, fed to UPGMA, reconstructs the expected (e.g. known correct) phylogenetic tree

## Limitations

- The distance metric reflects not only amino acid sequence but also protein abundance (or coverage) differences between samples
- Similarity of datasets with no similar spectra tends to 0 (distance +∞) as set size grows; the extremes of none or all spectra being similar are unlikely in practice
- Using all acquired data means heavily degraded proteins or proteins with a priori unknown modifications are included, which affects the metric

## Evidence

- [abstract] The distribution of spectra dot products for thousands of tandem mass spectra collected from two samples provides a measure on the fraction of shared peptides between the two samples.: "The distribution of spectra dot products for thousands of tandem mass spectra collected from two samples provides a measure on the fraction of shared peptides between the two samples."
- [readme] compareMS2 is a tool for direct comparison of tandem mass spectrometry datasets, typically from liquid chromatography-tandem mass spectrometry (LC-MS/MS), defining similarity as a function of shared (similar) spectra and distance as the inverse of this similarity.: [evidence span withheld — anchor status: external_doc]
- [readme] Each line should contain a filename and a species name, separated by a TAB character.: "Each line should contain a filename
and a species name, separated by a TAB character. If the directory with samples contains
a file with the exact name "spectra_to_species.txt", that file is"
- [readme] The compareMS2 GUI provides real-time feedback by continuously updating the distance matrix, and drawing a UPGMA tree at the completion of each row in the (lower triangular) distance matrix.: [evidence span withheld — anchor status: external_doc]
