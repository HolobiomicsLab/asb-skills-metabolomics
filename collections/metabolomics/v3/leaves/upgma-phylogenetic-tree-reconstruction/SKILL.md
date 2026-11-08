---
name: upgma-phylogenetic-tree-reconstruction
description: Use when you have pairwise spectral distances between tandem mass spectrometry
  datasets (e.g. from compareMS2 on MGF files) and need to reconstruct the phylogenetic
  relatedness of species, proteomes, or samples from those distances.
when_to_use_negative:
- The goal is species identification of a single unknown sample against a reference
  collection — use compareMS2's spectra2species mode instead, which produces a bar
  chart of distances rather than a tree.
- You want an overview of charge states, noise, or mass accuracy within or between
  two files — use the compareMS2 heatmap visualization, not UPGMA clustering.
- Input spectra require protein identification before comparison (database search,
  spectral library, or de novo sequencing) — compareMS2 compares tandem mass spectra
  directly without sequence data, so this skill is wrong when a Mascot/SpectraST-style
  identification workflow is the actual requirement.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0121
tools:
- name: compareMS2 2.0
  role: Computes pairwise spectral similarity/distance between datasets and applies
    UPGMA clustering, drawing the tree with a quality metric in its GUI and exporting
    Newick output.
  repo: https://github.com/524D/compareMS2
- name: MEGA11
  role: 'Independent cross-check: the distance matrix is imported into MEGA11 to reconstruct
    the phylogenetic tree and confirm the topology obtained in compareMS2.'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:4daf2ce449d72cc818e01ea97ac3f2114b83afcd1533a77ff5bbf18f009eb166
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2
  tools:
  - MEGA11
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# upgma-phylogenetic-tree-reconstruction

## Summary

Builds a molecular phylogenetic tree by converting the symmetric distance matrix produced from pairwise LC-MS/MS tandem mass spectra comparisons (via compareMS2 2.0) into a UPGMA dendrogram, which the compareMS2 2.0 GUI displays with a quality metric and can export in Newick format.

## When to use

Use when you have pairwise spectral distances between tandem mass spectrometry datasets (e.g. from compareMS2 on MGF files) and need to reconstruct the phylogenetic relatedness of species, proteomes, or samples from those distances.

## When NOT to use

- The goal is species identification of a single unknown sample against a reference collection — use compareMS2's spectra2species mode instead, which produces a bar chart of distances rather than a tree.
- You want an overview of charge states, noise, or mass accuracy within or between two files — use the compareMS2 heatmap visualization, not UPGMA clustering.
- Input spectra require protein identification before comparison (database search, spectral library, or de novo sequencing) — compareMS2 compares tandem mass spectra directly without sequence data, so this skill is wrong when a Mascot/SpectraST-style identification workflow is the actual requirement.

## Inputs

- Symmetric distance matrix derived from all pairwise compareMS2 2.0 comparisons of LC-MS/MS datasets (MGF files with tandem mass spectra)
- Set of MGF files (e.g. five serum datasets from PXD034932, 42,629–47,626 tandem mass spectra each)
- Optional sample-to-species mapping file ('spectra_to_species.txt', TAB-separated filename/species lines)

## Outputs

- UPGMA phylogenetic tree with a tree-quality metric, rendered by the compareMS2 2.0 GUI
- Tree exported in Newick format
- Symmetric distance matrix (lower-triangular, updated continuously during the run)

## How to apply

First obtain the symmetric distance matrix by running compareMS2 2.0 on every pair of MGF datasets (e.g. the five new serum datasets from PXD034932, each containing 42,629–47,626 tandem mass spectra) with the paper's default parameters: precursor mass difference 2.05 and spectral similarity score cutoff 0.8. The compareMS2 GUI continuously updates the distance matrix and draws a UPGMA tree at the completion of each row in the lower-triangular matrix, giving real-time feedback. Apply UPGMA hierarchical clustering to assemble the pairwise distances into the phylogenetic tree, using the tree-quality metric displayed by the compareMS2 2.0 GUI to judge the topology. Optionally cross-check the topology by importing the distance matrix into MEGA11, as the paper's authors did to reconstruct the correct tree for the serum datasets. Export the tree in Newick format; the GUI supports pausing or terminating execution at any stage and offers circular rendering as shown for the primate sera test data.

## Related tools

- **compareMS2 2.0** (Computes pairwise spectral similarity/distance between datasets and applies UPGMA clustering, drawing the tree with a quality metric in its GUI and exporting Newick output.) — https://github.com/524D/compareMS2
- **MEGA11** (Independent cross-check: the distance matrix is imported into MEGA11 to reconstruct the phylogenetic tree and confirm the topology obtained in compareMS2.)

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- The reconstructed UPGMA tree matches the correct phylogenetic topology reported for the PXD034932 serum datasets (California sea lion, dog, rock hyrax, white-tailed deer) in both compareMS2 and MEGA11.
- The distance matrix is symmetric and lower-triangular: D(A,B) equals D(B,A), and diagonal entries are zero, so only one pairwise direction per dataset pair is computed.
- The tree-quality metric shown by the compareMS2 2.0 GUI is inspected after clustering completes.
- Identical-input invariant: a dataset compared to itself yields distance 0, and the GUI renders the UPGMA tree after each row of the lower-triangular distance matrix finishes.
- All 10 pairwise comparisons for five datasets complete with the default parameters (precursor mass difference 2.05, spectral similarity score cutoff 0.8), taking about 40 minutes on the reported Intel Xeon W-2135 desktop.

## Limitations

- The distance metric depends not only on peptide amino acid sequences but also on protein abundance (or coverage), so datasets from the same species with differing proteome coverage may cluster unexpectedly.
- Comparing many datasets is computationally heavy: 100 LC-MS/MS datasets require 4,950 pairwise comparisons taking several hours (only 5 datasets/10 pairs took ~40 min with default parameters).
- The GUI's NodeJS build currently cannot use the latest NodeJS version due to an Electron Forge bug (June 2026 note in README); use NodeJS v22.22.3 as instructed.

## Evidence

- [results] UPGMA tree from symmetric spectral distance matrix with GUI quality metric: "The compareMS2 2.0 GUI (Figure 2) displays a phylogenetic tree with a quality metric"
- [results] Default compareMS2 parameters for the serum datasets: [evidence span withheld — anchor status: not_found]
- [readme] Distance metric is symmetric: [evidence span withheld — anchor status: external_doc]
- [readme] GUI draws UPGMA tree as distance matrix rows complete: [evidence span withheld — anchor status: external_doc]
- [readme] Newick export supported by the GUI: [evidence span withheld — anchor status: external_doc]
- [readme] Distance reflects abundance as well as sequence: [evidence span withheld — anchor status: external_doc]
