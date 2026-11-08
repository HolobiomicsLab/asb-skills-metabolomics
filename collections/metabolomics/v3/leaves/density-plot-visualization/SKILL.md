---
name: density-plot-visualization
description: 'Use when comparing chemical-space coverage of two or more MGF-format
  spectral libraries: parse SMILES annotations, build a union of unique structures,
  embed with UMAP (n_neighbors=5000, min_dist=0.5), and shade per-library density.'
when_to_use_negative:
- Input is already a feature/embedding table of structures — skip re-parsing and go
  straight to plotting, but this skill's parsing-and-union pipeline is redundant
- The goal is quantitative spectral similarity scoring (cosine or spectral entropy)
  rather than qualitative chemical-space coverage comparison
- Only one library is available — per-library density shading requires at least two
  libraries to show overlap
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: umap-learn
  role: Computes the 2D UMAP embedding of standardized SMILES structures with n_neighbors=5000
    and min_dist=0.5 prior to density rendering
- name: matchms
  role: Imports and cleans MGF spectral libraries and their SMILES metadata before
    the union table is built
  repo: https://github.com/matchms/matchms
- name: Python
  role: Implementation language for parsing MGF metadata, standardizing SMILES, embedding,
    and rendering the density plot
- name: pip
  role: Package manager used to install dependencies (e.g. pip install -e . in the
    spectraverse-analysis repo)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:d0d528d6bf5d94f0481730a316728a13a82a92681839bd0d8cd8fb5099f2d259
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matchms/matchms
  tools:
  - Python
  - conda
  - matchms
  - pip
  - umap-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# density-plot-visualization

## Summary

Renders a 2D UMAP kernel-density visualization of chemical-space coverage for MS/MS spectral libraries (Spectraverse, MassSpecGym, NPLIB1), showing per-library density over a shared structural embedding to demonstrate Spectraverse's broader coverage.

## When to use

Use when comparing chemical-space coverage of two or more MGF-format spectral libraries: parse SMILES annotations, build a union of unique structures, embed with UMAP (n_neighbors=5000, min_dist=0.5), and shade per-library density.

## When NOT to use

- Input is already a feature/embedding table of structures — skip re-parsing and go straight to plotting, but this skill's parsing-and-union pipeline is redundant
- The goal is quantitative spectral similarity scoring (cosine or spectral entropy) rather than qualitative chemical-space coverage comparison
- Only one library is available — per-library density shading requires at least two libraries to show overlap

## Inputs

- MGF-format spectral library files with SMILES metadata (Spectraverse Zenodo deposit, MassSpecGym.mgf, NPLIB1 Zenodo 8316682 archive)
- Union table of standardized, unique chemical structures with per-library presence flags

## Outputs

- umap_density_2d.png (2D kernel-density plot per library over shared UMAP embedding)
- CSV coordinate table: one row per unique structure with x/y UMAP coordinates and library membership annotations

## How to apply

Load the three MGF-format libraries (Spectraverse Zenodo deposit v1.0.1, MassSpecGym.mgf from HuggingFace, NPLIB1 archive from Zenodo doi:10.5281/zenodo.8316682) and parse SMILES structure annotations from spectrum metadata using Python. Standardize the SMILES and collapse them into a single union table of unique structures, keeping a per-library presence flag per structure. Compute a 2D UMAP embedding with umap-learn using the paper's parameters n_neighbors=5000 and min_dist=0.5, then render a per-library kernel-density shading over the embedding and save it as umap_density_2d.png. Export the 2D UMAP coordinates for every unique structure with library-membership annotations to a CSV coordinate table. Rationale: density over a shared embedding, rather than side-by-side embeddings, makes overlap and differential coverage (Spectraverse's broader chemical space) directly visible.

## Related tools

- **umap-learn** (Computes the 2D UMAP embedding of standardized SMILES structures with n_neighbors=5000 and min_dist=0.5 prior to density rendering)
- **matchms** (Imports and cleans MGF spectral libraries and their SMILES metadata before the union table is built) — https://github.com/matchms/matchms
- **Python** (Implementation language for parsing MGF metadata, standardizing SMILES, embedding, and rendering the density plot)
- **pip** (Package manager used to install dependencies (e.g. pip install -e . in the spectraverse-analysis repo))

## Examples

```
import umap; emb = umap.UMAP(n_neighbors=5000, min_dist=0.5).fit_transform(fingerprints)
```

## Evaluation signals

- umap_density_2d.png exists and shows per-library kernel-density shading over the shared UMAP embedding
- Coordinate CSV contains exactly one row per unique structure in the SMILES union, with x/y columns populated and library-membership flags
- UMAP hyperparameters match the paper's specification: n_neighbors=5000 and min_dist=0.5
- Union table size is consistent with library contents (Spectraverse: 44,237 unique small molecules; MassSpecGym ~250,000 fewer spectra than Spectraverse)
- Density visualization qualitatively reproduces the paper's result that Spectraverse covers broader chemical space than MassSpecGym and NPLIB1

## Limitations

- UMAP embeddings are stochastic; without a fixed random seed, coordinates (though not the qualitative coverage conclusion) vary between runs
- The result is a qualitative density visualization, not a quantitative coverage metric
- n_neighbors=5000 is tuned for library-scale data (~tens of thousands of structures); it will behave differently on much smaller inputs
- Spectraverse's de-duplication completeness is not fully stated in the paper (near-duplicate removal was reported for MassSpecGym comparisons), so overlapping structures between libraries could affect union size

## Evidence

- [other] Compute a 2D UMAP embedding of the structures with umap-learn using n_neighbors=5000 and min_dist=0.5.: [evidence span withheld — anchor status: not_found]
- [abstract] Broader coverage of chemical space afforded by Spectraverse: "The resulting resource affords the most comprehensive coverage of chemical space of any machine-learning-ready library of MS/MS spectra to date"
- [readme] Spectraverse-analysis repo contents: "This repository contains code used to preprocess and harmonize MS/MS spectra and their associated metadata in the compilation of Spectraverse."
- [readme] matchms supports MGF import: [evidence span withheld — anchor status: external_doc]
