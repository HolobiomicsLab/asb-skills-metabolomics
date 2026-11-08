---
name: tandem-ms-pairwise-distance-computation
description: Use when you need pairwise distances between multiple LC-MS/MS tandem
  mass spectra datasets without prior peptide identification, e.g. for phylogenetic
  trees, batch-effect/outlier QC, or species identification.
when_to_use_negative:
- When peptide identifications or spectral libraries are available and you want peptide-level
  analysis (use database search like Mascot/Comet or spectral library search like
  SpectraST/BiblioSpec instead of direct comparison)
- When you need de novo peptide sequence from spectra lacking any genome or library
  reference (use de novo tools such as PepNovo/LUTEFISK)
- When comparing fewer than two datasets, or when input data are not tandem mass spectra
  (e.g. profile MS1 or feature tables)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: compareMS2 2.0
  role: Primary tool that computes the spectral similarity (eq 1) and pairwise distance
    matrix between tandem mass spectra datasets via CLI or GUI
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: Downstream reference for reconstructing the phylogenetic tree from the distance
    matrix and validating the tree topology
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:cb20ac38b9eb48fe858d16b520b98c7aca4acf3a4388788d3849937f50f09033
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - Intel Xeon W-2135
  - MEGA11
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tandem-ms-pairwise-distance-computation

## Summary

Compute pairwise distances between sets of tandem mass spectra (e.g. LC-MS/MS MGF datasets) using compareMS2 2.0, which defines similarity as a function of shared/similar spectra and produces a symmetric distance matrix for phylogenetics, QC, or species identification.

## When to use

Use when you need pairwise distances between multiple LC-MS/MS tandem mass spectra datasets without prior peptide identification, e.g. for phylogenetic trees, batch-effect/outlier QC, or species identification.

## When NOT to use

- When peptide identifications or spectral libraries are available and you want peptide-level analysis (use database search like Mascot/Comet or spectral library search like SpectraST/BiblioSpec instead of direct comparison)
- When you need de novo peptide sequence from spectra lacking any genome or library reference (use de novo tools such as PepNovo/LUTEFISK)
- When comparing fewer than two datasets, or when input data are not tandem mass spectra (e.g. profile MS1 or feature tables)

## Inputs

- Sets of tandem mass spectra (LC-MS/MS MGF files), e.g. five serum datasets from ProteomeXchange PXD034932 containing 42,629–47,626 spectra each
- Optional sample-to-species mapping file ('spectra_to_species.txt' with filename<TAB>species per line) for species-name display and averaging per species

## Outputs

- Lower-triangular pairwise distance matrix between all input MS2 datasets
- UPGMA phylogenetic tree (GUI mode) with quality metric
- spectra2species bar chart (one file vs. a collection) or spectral-inner-product heatmap in alternative compare modes
- Wall-clock runtime measurements for the batch of pairwise comparisons

## How to apply

Clone the 524D/compareMS2 repository and install it via `corepack enable` then `yarn` (NodeJS 22.22.3; later versions have an Electron Forge bug as of June 2026). Run compareMS2 2.0 in command-line or GUI mode with default parameters over your MGF datasets: every pairwise comparison D(A,B) of tandem mass spectra sets is computed only once because the default distance metric is fully symmetric, so D(B,A) is skipped and D(A,A) is always zero. The similarity is calculated from the spectral overlap in eq 1, driven by m/z windows and spectral similarity thresholds that can be estimated beforehand from the spectral similarity as a function of precursor m/z difference. The GUI continuously updates the distance matrix and draws a UPGMA phylogenetic tree at completion of each lower-triangular row; executions can be paused or terminated at any stage. Timing matters for benchmarking: the 10 pairwise comparisons over five serum datasets (42,629–47,626 spectra each) took 40 min on an Intel Xeon W-2135 at 3.70 GHz.

## Related tools

- **compareMS2 2.0** (Primary tool that computes the spectral similarity (eq 1) and pairwise distance matrix between tandem mass spectra datasets via CLI or GUI) — https://github.com/524D/compareMS2.git
- **MEGA11** (Downstream reference for reconstructing the phylogenetic tree from the distance matrix and validating the tree topology)

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- Total wall-clock time for the 10 pairwise comparisons over five serum datasets should be same order of magnitude as the reported 40 min with default parameters on an Intel Xeon W-2135 (3.70 GHz)
- The distance matrix should be symmetric — verify D(A,B) equals D(B,A) and diagonal entries are zero, so only n(n−1)/2 comparisons run (10 for five datasets, 4950 for 100 datasets)
- Datasets with identical spectral content must yield similarity 1 (distance 0); datasets with no similar spectra should trend toward distance +∞ as set size grows
- Phylogenetic tree topology reconstructed in compareMS2 should match that reconstructed in MEGA11 from the same distance matrix
- Runtime should scale with dataset size (number of tandem mass spectra per file, 42,629–47,626 here) and number of pairwise comparisons

## Limitations

- Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical
- The distance metric is also affected by protein abundance or coverage, not just peptide sequence
- The paper reports 40 min on one specific CPU (Intel Xeon W-2135, 3.70 GHz); runtimes will differ by hardware and cannot be assumed to transfer exactly
- The default symmetric metric halves the number of comparisons, but per-pair spectra-similarity scoring is CPU-bound and grows with dataset size

## Evidence

- [results] The 10 pairwise comparisons in compareMS2 took 40 min with default parameters: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [readme] Default metric is symmetric, skip redundant comparisons: [evidence span withheld — anchor status: external_doc]
- [readme] Datasets with identical spectral content yield similarity 1, distance 0: [evidence span withheld — anchor status: external_doc]
- [results] Jaccard similarity is sensitive to the spectrum-identity definition: "making the Jaccard similarity very sensitive to the precise definition of when two spectra are considered identical"
- [methods] Clone and install procedure: "git clone https://github.com/524D/compareMS2.git
cd compareMS2
corepack enable
yarn"
