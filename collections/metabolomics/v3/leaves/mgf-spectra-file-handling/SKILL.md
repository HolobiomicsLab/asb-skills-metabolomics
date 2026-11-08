---
name: mgf-spectra-file-handling
description: Use when LC-MS/MS acquisitions (e.g. serum or food/feed samples) are
  exported as MGF files and the goal is direct tandem-spectrum comparison — phylogenetics,
  species identification, or dataset QC — before any database search.
when_to_use_negative:
- When sequence databases or spectral libraries are available and the goal is peptide/protein
  identification — that is the database-search or spectral-library-search branch of
  the tool taxonomy, not direct spectral comparison.
- When the input is already processed (e.g. a feature table or identified-peptide
  list) rather than raw MGF tandem spectra — compareMS2 operates on the spectral content
  itself.
- When only a single MGF file is available and no reference set exists for spectra2species
  or phylogenetic comparison.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0080
tools:
- name: compareMS2
  role: Reads the MGF tandem spectra files, computes pairwise spectral-similarity
    distances, and produces phylogenetic trees, spectra2species bar charts, or heatmaps.
  repo: https://github.com/524D/compareMS2.git
- name: compareMS2 2.0
  role: Updated version with a GUI that controls MGF file selection, instrument-specific
    settings, real-time distance-matrix updates, and UPGMA tree display.
  repo: https://github.com/524D/compareMS2.git
- name: MEGA11
  role: 'Optional downstream validator: imports the distance matrix derived from the
    MGF comparison to independently reconstruct and cross-check the phylogenetic tree.'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00457
    title: 'compareMS2 2.0: An Improved Software for Comparing Tandem Mass Spectrometry
      Datasets'
  source_content_hash: sha256:3f2ce10e6ec3b664f2ad64760cb37458dbefb4fb609035cf73a9d7cce7446ff1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/524D/compareMS2.git
  tools:
  - MEGA11
  - compareMS2
  - compareMS2 2.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mgf-spectra-file-handling

## Summary

Prepare and supply MGF-format tandem mass spectrometry datasets as direct input to compareMS2 2.0, where pairwise spectral similarity is computed without prior peptide identification. Correct MGF handling is a prerequisite for reconstructing phylogenetic trees and QC/heatmap visualizations from LC-MS/MS data.

## When to use

Use when LC-MS/MS acquisitions (e.g. serum or food/feed samples) are exported as MGF files and the goal is direct tandem-spectrum comparison — phylogenetics, species identification, or dataset QC — before any database search.

## When NOT to use

- When sequence databases or spectral libraries are available and the goal is peptide/protein identification — that is the database-search or spectral-library-search branch of the tool taxonomy, not direct spectral comparison.
- When the input is already processed (e.g. a feature table or identified-peptide list) rather than raw MGF tandem spectra — compareMS2 operates on the spectral content itself.
- When only a single MGF file is available and no reference set exists for spectra2species or phylogenetic comparison.

## Inputs

- MGF files of tandem mass spectra, one per LC-MS/MS dataset (e.g. 42,629–47,626 spectra per serum sample)
- Optional spectra_to_species.txt: plain-text file, each line 'filename<TAB>species name'
- MS2 data files in a common directory for GUI selection

## Outputs

- Pairwise distance values between MGF datasets (e.g. 10 comparisons for 5 serum datasets, ~40 min on an Intel Xeon W-2135 at 3.70 GHz)
- Symmetric distance matrix rendered live in the compareMS2 GUI
- UPGMA phylogenetic tree (with quality metric), spectra2species bar chart, or spectral inner-product heatmap
- Tree exportable in Newick format for external tools such as MEGA11

## How to apply

Export each LC-MS/MS acquisition as an MGF file containing its tandem mass spectra; for the paper's serum validation, each of the five new serum datasets contained between 42,629 and 47,626 tandem mass spectra. Feed the MGF files into compareMS2 2.0, which compares the spectral sets directly (not spectra-to-predicted-peptide), using the GUI's phylogenetic-tree, spectra2species, or heatmap modes depending on the analysis. Optionally supply a tab-separated 'spectra_to_species.txt' mapping file (filename<TAB>species name) so the GUI displays species names and averages multiple MGF files per species; if a file with that exact name sits in the sample directory it is auto-selected. All pairwise comparisons among n MGF files form a symmetric distance matrix that is then clustered (e.g. UPGMA) into a tree; validation used PRIDE dataset PXD034932, where the correct tree topology was recovered in both compareMS2 and MEGA11.

## Related tools

- **compareMS2** (Reads the MGF tandem spectra files, computes pairwise spectral-similarity distances, and produces phylogenetic trees, spectra2species bar charts, or heatmaps.) — https://github.com/524D/compareMS2.git
- **compareMS2 2.0** (Updated version with a GUI that controls MGF file selection, instrument-specific settings, real-time distance-matrix updates, and UPGMA tree display.) — https://github.com/524D/compareMS2.git
- **MEGA11** (Optional downstream validator: imports the distance matrix derived from the MGF comparison to independently reconstruct and cross-check the phylogenetic tree.)

## Examples

```
git clone https://github.com/524D/compareMS2.git && cd compareMS2 && corepack enable && yarn && yarn start
```

## Evaluation signals

- Each input MGF must contain the expected number of tandem spectra (paper's serum datasets: 42,629–47,626).
- The pairwise distance matrix must be symmetric and have zero diagonal (D(A,B) = D(B,A); D(A,A) = 0), so only the lower triangle needs computing.
- For the PXD034932 serum test set, the UPGMA tree built from the MGF-derived distances must match the correct phylogenetic topology recoverable in both compareMS2 and MEGA11.
- The spectra_to_species.txt file must be tab-separated with one filename–species pair per line; a file of exactly that name in the sample directory is auto-selected.
- The 10 pairwise comparisons of five serum datasets completed in about 40 minutes with default parameters on an Intel Xeon W-2135 (3.70 GHz) — large deviations suggest misconfigured MGF input or wrong file set.

## Limitations

- MGF files are the expected input format; data must be exported/converted to MGF before compareMS2 can consume them.
- Distance depends not only on peptide amino-acid sequences but also on protein abundance (or coverage), so MGF comparisons reflect proteome composition, not sequence alone.
- Jaccard similarity is very sensitive to the precise definition of when two spectra are considered identical.
- Spectral clustering on this data tends to produce large globs of spectra rather than a distinct cluster per peptide.
- Currently (June 2026) the latest NodeJS version cannot be used to build/run the tool due to a bug, so environment setup for GUI-based MGF handling must pin NodeJS 22.22.3.

## Evidence

- [readme] MGF files as the working input for phylogenetic reconstruction: [evidence span withheld — anchor status: external_doc]
- [results] Dataset sizes used for validation: "each containing between 42,629 and 47,626 tandem mass spectra"
- [discussion] Direct spectral comparison, no prior identification needed: "compareMS2 compares sets of tandem mass spectra to each other rather than to predicted spectra of specific peptides as when identifying proteins from tandem mass spectra."
- [readme] spectra_to_species.txt auto-selection rule: [evidence span withheld — anchor status: external_doc]
- [readme] Symmetric distance metric and real-time UPGMA: [evidence span withheld — anchor status: external_doc]
- [readme] Reference MGF test data availability: [evidence span withheld — anchor status: not_found]
- [results] Jaccard sensitivity caveat: "making the Jaccard similarity very sensitive to the precise definition of when two spectra are considered identical"
