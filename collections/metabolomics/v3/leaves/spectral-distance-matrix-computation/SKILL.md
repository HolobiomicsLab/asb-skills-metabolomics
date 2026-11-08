---
name: spectral-distance-matrix-computation
description: Use when you have two or more LC-MS/MS datasets (e.g. MGF files with
  tens of thousands of tandem mass spectra) and need quantitative pairwise spectral
  distances — e.
when_to_use_negative:
- When reference genome or protein sequences are available and a traditional database
  search (Mascot, Comet, Sage) or spectral library search (BiblioSpec, SpectraST)
  would assign peptides directly.
- When the goal is peptide identification rather than dataset-level similarity or
  phylogenetic relatedness — compareMS2 compares sets of tandem mass spectra directly,
  not predicted spectra of specific peptides.
- When the task is pairwise peptide-level annotation, e.g. estimating posterior error
  probabilities via PeptideProphet or Percolator; those operate on identifications,
  not raw spectral overlap.
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3174
tools:
- name: compareMS2 2.0
  role: 'Primary tool: runs all pairwise spectral comparisons with default parameters
    (precursor mass difference 2.05, similarity cutoff 0.8) and assembles the symmetric
    distance matrix with real-time GUI updates'
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: 'External validation: import the computed distance matrix to independently
    reconstruct and confirm the phylogenetic tree topology'
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:c8bd79effc80a655ad002c324b5f146b65c1320c0abb31543790134b9116f85c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - MEGA11
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-distance-matrix-computation

## Summary

Computes a pairwise global distance matrix over a set of tandem mass spectrometry (LC-MS/MS) datasets by directly comparing their tandem mass spectra with compareMS2 2.0, without any prior protein identification or spectral library, and assembles the pairwise distances into a symmetric distance matrix suitable for downstream clustering or phylogenetics.

## When to use

Use when you have two or more LC-MS/MS datasets (e.g. MGF files with tens of thousands of tandem mass spectra) and need quantitative pairwise spectral distances — e.g. for molecular phylogenetics, food/feed species identification, sample-similarity heatmaps, spectra2species bar charts, or detecting outliers and batch effects before database search — but you lack reference sequence databases or peptide-to-spectrum assignments.

## When NOT to use

- When reference genome or protein sequences are available and a traditional database search (Mascot, Comet, Sage) or spectral library search (BiblioSpec, SpectraST) would assign peptides directly.
- When the goal is peptide identification rather than dataset-level similarity or phylogenetic relatedness — compareMS2 compares sets of tandem mass spectra directly, not predicted spectra of specific peptides.
- When the task is pairwise peptide-level annotation, e.g. estimating posterior error probabilities via PeptideProphet or Percolator; those operate on identifications, not raw spectral overlap.

## Inputs

- MGF-format tandem mass spectrometry (LC-MS/MS) dataset files, e.g. serum datasets with 42,629–47,626 spectra each
- Two or more dataset files to compare pairwise (N datasets → N(N-1)/2 unique comparisons)
- Optional plain-text 'spectra_to_species.txt' sample-to-species mapping (filename<TAB>species per line)

## Outputs

- Pairwise spectral distance values for each unique dataset pair
- Fully symmetric N×N distance matrix (D(A,B) = D(B,A), D(A,A) = 0), e.g. saved as distance_matrix.tsv
- Lower-triangular distance matrix assembled by the GUI, updated in real time during the run
- UPGMA phylogenetic tree rendered from the completed distance matrix (Newick-exportable)
- Spectra2species bar chart (one file vs. a collection) or self-similarity heatmap when alternate GUI modes are selected

## How to apply

Install compareMS2 2.0 by cloning https://github.com/524D/compareMS2.git, enabling yarn via corepack, and running `yarn`; or use the Windows/Ubuntu installer from the releases page. Provide input files in MGF format — e.g. the five serum datasets from ProteomeXchange PXD034932, each containing between 42,629 and 47,626 tandem mass spectra. Run compareMS2 2.0 on every pair of datasets under the default parameters (precursor mass difference 2.05, spectral similarity score cutoff 0.8), which uses the spectral similarity in eq 1 to calculate the overlap between each pair of spectrum sets. Only compute each unique comparison once: the distance metric D is fully symmetric, so D(A,B) = D(B,A) and D(A,A) = 0, meaning N datasets require N(N-1)/2 unique pairwise comparisons (e.g. 5 datasets → 10 comparisons). If a 'spectra_to_species.txt' sample-to-species file is present, GUI displays use species names instead of file names and average results across multiple sample files per species. Collect the pairwise distances into a lower-triangular matrix, mirror it into a fully symmetric 5×5 (or N×N) matrix, and save it for downstream UPGMA tree-building, heatmap generation, or import into MEGA11. Optionally cross-check topology by loading the distance matrix into MEGA11.

## Related tools

- **compareMS2 2.0** (Primary tool: runs all pairwise spectral comparisons with default parameters (precursor mass difference 2.05, similarity cutoff 0.8) and assembles the symmetric distance matrix with real-time GUI updates) — https://github.com/524D/compareMS2.git
- **MEGA11** (External validation: import the computed distance matrix to independently reconstruct and confirm the phylogenetic tree topology)

## Examples

```
git clone https://github.com/524D/compareMS2.git; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- The assembled distance matrix must be exactly symmetric: D(A,B) = D(B,A) for every pair, and all diagonal entries must equal zero.
- For N datasets, exactly N(N-1)/2 unique pairwise comparisons must be reported — e.g. 5 serum datasets must yield 10 unique distances (the 10 pairwise comparisons that took 40 min on the reference Intel Xeon W-2135 CPU at 3.70 GHz).
- When applied to the five PXD034932 serum datasets (California sea lion, dog, rock hyrax, white-tailed deer), the resulting matrix must reproduce the correct phylogenetic tree topology the paper reports recovering in compareMS2 and MEGA11.
- Datasets with identical spectral content must yield distance 0 (similarity 1); datasets with no similar spectra must trend toward distance ∞ (similarity → 0) as set size grows.
- If a sample-to-species file is used, the GUI must display species names instead of file names, and results from multiple sample files of the same species must be averaged.

## Limitations

- Runtime scales quadratically: 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours even with default parameters.
- The Jaccard similarity underlying the metric is very sensitive to the precise definition of when two spectra are considered identical.
- Not limited to peptide tandem mass spectra but distance is also affected by protein abundance/coverage, so distance reflects both sequence divergence and expression differences, not only evolutionary relatedness.
- Requires NodeJS 22.x (a specific version; the latest NodeJS could not be used due to a bug in electron-forge) and yarn/corepack setup; Windows install of corepack must be done from an administrator command prompt.

## Evidence

- [results] each containing between 42,629 and 47,626 tandem mass spectra: "Using the five new serum datasets, each containing between 42,629 and 47,626 tandem mass spectra"
- [results] 10 pairwise comparisons in compareMS2 took 40 min with default parameters: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [results] correct phylogenetic tree reconstructed in compareMS2 and MEGA11: "we could reconstruct the correct phylogenetic tree in compareMS2 and MEGA11"
- [readme] compareMS2 is a tool for direct comparison of tandem mass spectrometry datasets: [evidence span withheld — anchor status: external_doc]
- [readme] The default distance metric D is symmetric, i.e. the distance from dataset A to dataset B is identical to the distance from dataset B to dataset A: [evidence span withheld — anchor status: external_doc]
- [methods] git clone https://github.com/524D/compareMS2.git: "git clone https://github.com/524D/compareMS2.git
cd compareMS2
corepack enable
yarn"
- [results] comparing 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours: "comparing 100 LC-MS/MS datasets require 4950 pairwise comparisons, taking several hours"
- [results] uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra: "uses the spectral similarity in eq 1 to calculate the overlap between sets of tandem mass spectra"
- [readme] Each line should contain a filename and a species name, separated by a TAB character: [evidence span withheld — anchor status: external_doc]
- [results] making the Jaccard similarity very sensitive to the precise definition of when two spectra are considered identical: "making the Jaccard similarity very sensitive to the precise definition of when two spectra are considered identical"
