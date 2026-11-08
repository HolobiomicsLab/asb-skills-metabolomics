---
name: tandem-ms-dataset-pairwise-comparison
description: Use when you have raw LC-MS/MS dataset files (e.g. MGF with tens of thousands
  of tandem spectra) and need dataset-level similarity, species-level phylogenetic
  reconstruction, or QC/batch-effect detection before any database search.
when_to_use_negative:
- When the goal is peptide or protein identification — compareMS2 compares sets of
  tandem mass spectra to each other rather than to predicted spectra of specific peptides
  as when identifying proteins from tandem mass spectra
- When spectra-to-peptide cluster structure is needed — spectral clustering tends
  to produce large globs of spectra rather than a distinct cluster for each peptide,
  making results sensitive to the precise definition of when two spectra are considered
  identical
- When datasets have no overlapping spectral content at all — similarity tends to
  0 (distance +∞) as set size grows, giving no discriminative information
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3292
tools:
- name: compareMS2 2.0
  role: 'Primary tool: computes all pairwise distances between tandem MS datasets
    and renders the UPGMA tree, spectra2species bar chart and heatmap in its GUI;
    also runnable headless via CLI with `yarn start`.'
  repo: https://github.com/524D/compareMS2
- name: MEGA11
  role: 'Independent verification: import the compareMS2 distance matrix to cross-check
    the reconstructed tree topology.'
- name: DISMS2
  role: Alternative library in the same 'direct comparison' class of tools (no prior
    sequence or spectral library needed), cited as similar work but not used in the
    paper's workflow.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:14b1c70e8336fae125eba3faed94328ae26b44adafd994592515dd071f22ad95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2
  tools:
  - DISMS2
  - MEGA11
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tandem-ms-dataset-pairwise-comparison

## Summary

Compute pairwise global-similarity distances between tandem MS/MS (MGF) datasets directly from spectra, assemble them into a symmetric distance matrix, and build a UPGMA phylogenetic tree — the core compareMS2 2.0 workflow for species identification, QC, and molecular phylogenetics.

## When to use

Use when you have raw LC-MS/MS dataset files (e.g. MGF with tens of thousands of tandem spectra) and need dataset-level similarity, species-level phylogenetic reconstruction, or QC/batch-effect detection before any database search.

## When NOT to use

- When the goal is peptide or protein identification — compareMS2 compares sets of tandem mass spectra to each other rather than to predicted spectra of specific peptides as when identifying proteins from tandem mass spectra
- When spectra-to-peptide cluster structure is needed — spectral clustering tends to produce large globs of spectra rather than a distinct cluster for each peptide, making results sensitive to the precise definition of when two spectra are considered identical
- When datasets have no overlapping spectral content at all — similarity tends to 0 (distance +∞) as set size grows, giving no discriminative information

## Inputs

- Tandem MS/MS dataset files (MGF format), each typically containing tens of thousands of tandem mass spectra (e.g. 42,629–47,626 spectra per serum dataset)
- Optional 'Sample to species file' (spectra_to_species.txt) mapping sample filenames to species names via TAB-separated lines
- A collection of datasets from known species/conditions (e.g. primate sera, California sea lion, dog, rock hyrax, white-tailed deer sera) for reference phylogeny or species identification

## Outputs

- Pairwise distance matrix between all input datasets (symmetric; lower-triangular form)
- UPGMA phylogenetic tree with quality metric, exported in Newick format
- spectra2species bar chart (single unknown dataset vs. a reference set)
- Heatmap of inner-product distributions (charge states, mass accuracy, spectral similarity)

## How to apply

Run compareMS2 2.0 on every pair of input datasets using the paper's default parameters — a precursor mass difference window of 2.05 and a spectral similarity score cutoff of 0.8 — producing pairwise distances based on the fully symmetric overlap metric (eq 1); for N datasets this is N(N−1)/2 comparisons (10 for 5 serum datasets, ~40 min on an Intel Xeon W-2135 at 3.70 GHz). Because the default metric is symmetric, only calculate D(A,B) once; D(A,A) is always zero. Assemble the pairwise distances into a lower-triangular distance matrix; the GUI continuously updates the matrix and draws a UPGMA tree after each row completes, so no separate tree-building step is needed. Optionally use a 'Sample to species file' (spectra_to_species.txt, TAB-separated filename→species pairs) so multiple files per species are averaged. Export the resulting tree in Newick format and cross-check the topology in MEGA11. The distance metric is peptide-sequence- and abundance-dependent, so proteome composition (not just species) influences the result.

## Related tools

- **compareMS2 2.0** (Primary tool: computes all pairwise distances between tandem MS datasets and renders the UPGMA tree, spectra2species bar chart and heatmap in its GUI; also runnable headless via CLI with `yarn start`.) — https://github.com/524D/compareMS2
- **MEGA11** (Independent verification: import the compareMS2 distance matrix to cross-check the reconstructed tree topology.)
- **DISMS2** (Alternative library in the same 'direct comparison' class of tools (no prior sequence or spectral library needed), cited as similar work but not used in the paper's workflow.)

## Examples

```
git clone https://github.com/524D/compareMS2; cd compareMS2; corepack enable; yarn; yarn start
```

## Evaluation signals

- The 10 pairwise comparisons for the five serum datasets (each 42,629–47,626 spectra) completed in ~40 min with default parameters on a 3.70 GHz Intel Xeon W-2135 — runtime should scale roughly quadratically with dataset count (4950 comparisons for 100 datasets).
- The reconstructed UPGMA tree must match the known phylogenetic topology (e.g. California sea lion, dog, rock hyrax, white-tailed deer sera cluster correctly) when cross-checked in MEGA11.
- Every dataset is identical to itself, so the diagonal of the distance matrix must be zero and the matrix must be symmetric: D(A,B) = D(B,A).
- Datasets with identical spectral content must yield similarity 1 / distance 0; totally disjoint sets yield similarity → 0 / distance → ∞.
- The compareMS2 2.0 GUI reports a tree-quality metric that should be acceptable for the intended topology; the Newick export should round-trip losslessly into MEGA11.

## Limitations

- The default distance metric depends on peptide sequence AND protein abundance/coverage, so proteomes from different cell lines or tissues of the same species may appear distant — the tree reflects proteome similarity, not purely evolutionary distance.
- Computational cost grows quadratically with dataset count: comparing 100 LC-MS/MS datasets requires 4950 pairwise comparisons, taking several hours.
- The Jaccard-like similarity is very sensitive to the precise definition of when two spectra are considered identical.
- Spectral clustering (as an alternative approach) tends to produce large globs of spectra rather than a distinct cluster per peptide.
- No explicit version pinning is given in the article text; use the latest compareMS2 release from the GitHub repo or the automatic Windows installer.

## Evidence

- [results] Applying the compareMS2 2.0 pipeline to the five new serum datasets with default parameters reconstructs the correct phylogenetic tree in both compareMS2 and MEGA11.: [evidence span withheld — anchor status: not_found]
- [results] 10 pairwise comparisons in compareMS2 took 40 min with default parameters: "The 10 pairwise comparisons in compareMS2 took 40 min with default parameters"
- [readme] The GUI provides real-time feedback by continuously updating the distance matrix, and drawing a UPGMA tree at the completion of each row in the (lower triangular) distance matrix.: [evidence span withheld — anchor status: external_doc]
- [readme] compareMS2 calculates the global similarity between tandem mass spectrometry datasets, defining similarity as a function of shared (similar) spectra and distance as the inverse of this similarity.: "compareMS2 calculates the global similarity between tandem mass spectrometry datasets."
- [readme] The 'Sample to species file' maps file names to species names, allowing multiple sample files for the same species whose results are then averaged.: [evidence span withheld — anchor status: external_doc]
- [readme] The default distance metric D is symmetric; if D(A, B) has already been calculated, there is no need to calculate D(B, A).: [evidence span withheld — anchor status: external_doc]
