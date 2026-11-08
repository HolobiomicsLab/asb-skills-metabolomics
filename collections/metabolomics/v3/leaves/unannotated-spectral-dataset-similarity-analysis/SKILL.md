---
name: unannotated-spectral-dataset-similarity-analysis
description: Use when you have peak-listed tandem mass spectra (e.g. MGF files) from
  two or more samples and need a quantitative distance matrix of peptide sharing without
  peptide identifications or a sequence database.
when_to_use_negative:
- You need peptide sequence identifications or quantification of specific proteins
  — compareMS2 deliberately skips database or spectral library searches and gives
  only a global dataset-level similarity
- You only have a small annotated subset of identified spectra rather than all acquired
  unannotated MS2 spectra — the method's power comes from using all acquired data
- You need to distinguish samples where only protein abundance/coverage differs from
  the same species — the distance metric is affected by both amino acid sequences
  and protein abundance
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0092
tools:
- name: compareMS2
  role: 'Primary tool: computes pairwise dot-product comparisons of unannotated tandem
    mass spectra, builds the distance matrix, and draws UPGMA trees, spectra2species
    bar charts, and heatmaps via CLI or GUI'
  repo: https://github.com/524D/compareMS2.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/rcm.6162
    title: Molecular phylogenetics by direct comparison of tandem mass spectra
  source_content_hash: sha256:c88cf4c789cfdc66f3d4982daf74bad7fe02ddc3b41781ad72b551b984b95de4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - compareMS2
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# unannotated-spectral-dataset-similarity-analysis

## Summary

Implements the compareMS2 approach of computing global similarity between tandem mass spectrometry (LC-MS/MS) datasets by comparing unannotated MS2 spectra directly via dot products, then inverting the shared-peptide fraction into a pairwise distance metric for phylogenetics, QC, or species identification.

## When to use

Use when you have peak-listed tandem mass spectra (e.g. MGF files) from two or more samples and need a quantitative distance matrix of peptide sharing without peptide identifications or a sequence database.

## When NOT to use

- You need peptide sequence identifications or quantification of specific proteins — compareMS2 deliberately skips database or spectral library searches and gives only a global dataset-level similarity
- You only have a small annotated subset of identified spectra rather than all acquired unannotated MS2 spectra — the method's power comes from using all acquired data
- You need to distinguish samples where only protein abundance/coverage differs from the same species — the distance metric is affected by both amino acid sequences and protein abundance

## Inputs

- Peak-listed tandem mass spectrometry (MS2) files, typically MGF, from LC-MS/MS runs of enzymatically (e.g. tryptic) digested proteins
- Optional 'Sample to species file' (spectra_to_species.txt): plain text, one filename and species name per line separated by a TAB character

## Outputs

- Symmetric sample-by-sample distance matrix (distance_matrix.csv) where each entry is the inverted shared-peptide fraction for a sample pair
- UPGMA phylogenetic tree built from the lower triangular distance matrix
- Heatmap of the inner-product (dot product) distribution between two files, indicating charge states, mass accuracy, and similarity

## How to apply

Load the peak-listed tandem mass spectra (MGF) files for each sample dataset. For each pair of samples, compute the dot product (cosine-type spectral similarity) between every pair of unannotated tandem MS2 spectra, one from each dataset, using all acquired spectra rather than a small annotated subset so that heavily degraded proteins or proteins with unknown modifications can still be used. Summarize each dot-product distribution as the fraction of shared peptides between the two samples. Invert this fraction to obtain the pairwise distance (identical spectral content gives similarity 1 and distance 0), and assemble all distances into a symmetric sample-by-sample distance matrix. The default distance metric is symmetric, so D(A,B) need only be computed once, and D(A,A) is always zero. Optionally average results over multiple individuals per species using a spectra_to_species.txt file mapping filenames (TAB-separated) to species names before tree reconstruction.

## Related tools

- **compareMS2** (Primary tool: computes pairwise dot-product comparisons of unannotated tandem mass spectra, builds the distance matrix, and draws UPGMA trees, spectra2species bar charts, and heatmaps via CLI or GUI) — https://github.com/524D/compareMS2.git

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- The output is a complete symmetric distance matrix covering every sample pair, with distances consistent with the computed shared-peptide fractions
- Identical or identical-spectral-content samples yield distance 0 (similarity 1)
- Datasets with no similar spectra yield distance tending to infinity (similarity tending to 0) as set size grows
- Phylogenetic reconstruction from primate sera digests (e.g. the OSF sg796 test set of 1,000 spectra per file) recovers the known correct tree for great apes and other primates when averaging over individuals per species
- The GUI continuously updates the lower triangular distance matrix and draws a UPGMA tree at completion of each row, allowing row-by-row sanity checking

## Limitations

- The distance metric is influenced by protein abundance (or coverage), not only amino acid sequence differences
- Datasets with no similar spectra only tend to distance infinity as set sizes go to infinity; exact zero similarity for finite datasets is not attained
- The extremes of none or all spectra being similar between two LC-MS/MS datasets are unlikely in practice, so distances are relative rather than absolute
- Requires peak-listed MS2 data (MGF); raw instrument files must be converted first

## Evidence

- [readme] compareMS2 definition and similarity/distance semantics: [evidence span withheld — anchor status: not_found]
- [readme] Symmetric default metric; only half the matrix computed: [evidence span withheld — anchor status: external_doc]
- [readme] Sample-to-species file format and per-species averaging: [evidence span withheld — anchor status: not_found]
- [intro] Tool category: direct comparison without sequence libraries: [evidence span withheld — anchor status: external_doc]
