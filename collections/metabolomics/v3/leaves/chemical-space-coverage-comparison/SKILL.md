---
name: chemical-space-coverage-comparison
description: Use when you have multiple MGF-format spectral libraries with SMILES
  structure annotations and need a visual, structure-based comparison of which library
  spans more chemical space.
when_to_use_negative:
- You are comparing libraries by spectral similarity (cosine or spectral-entropy scores)
  rather than by chemical-structure coverage of their SMILES annotations.
- The spectra lack parseable or standardizable SMILES structure annotations, so no
  structure-level union table can be built.
- The goal is de-duplication or near-duplicate removal of spectra, not cross-library
  coverage visualization.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
tools:
- name: umap-learn
  role: Computes the 2D UMAP embedding of unique structures with n_neighbors=5000
    and min_dist=0.5
- name: Python
  role: Implementation language for parsing MGF metadata, standardizing SMILES, embedding,
    and plotting (Python 3.10+)
  repo: https://github.com/skinniderlab/spectraverse-analysis
- name: matchms
  role: MGF parsing and spectrum metadata handling for the spectral libraries being
    compared
  repo: https://github.com/matchms/matchms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:4c85172850122185824b531e8a47c2c132f2f88cf0d18f697b165907f8f7e180
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/skinniderlab/spectraverse-analysis
  tools:
  - Python
  - conda
  - matchms
  - pip
  - umap-learn
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# chemical-space-coverage-comparison

## Summary

Compares the chemical-space coverage of machine-learning-ready MS/MS spectral libraries (Spectraverse, MassSpecGym, NPLIB1) by embedding their unique structure annotations (SMILES) into a shared 2D UMAP space and rendering per-library density overlays, demonstrating Spectraverse's broader coverage.

## When to use

Use when you have multiple MGF-format spectral libraries with SMILES structure annotations and need a visual, structure-based comparison of which library spans more chemical space.

## When NOT to use

- You are comparing libraries by spectral similarity (cosine or spectral-entropy scores) rather than by chemical-structure coverage of their SMILES annotations.
- The spectra lack parseable or standardizable SMILES structure annotations, so no structure-level union table can be built.
- The goal is de-duplication or near-duplicate removal of spectra, not cross-library coverage visualization.

## Inputs

- Spectraverse MGF-format spectral library (Zenodo doi:10.5281/zenodo.17870921, v1.0.1)
- MassSpecGym.mgf spectral library (HuggingFace)
- NPLIB1 spectral library archive (Zenodo doi:10.5281/zenodo.8316682)
- SMILES structure annotations parsed from spectrum metadata in each MGF

## Outputs

- umap_density_2d.png: 2D UMAP density plot with per-library kernel-density shading
- CSV coordinate table: one row per unique structure in the union, with x/y UMAP coordinates and library membership flags

## How to apply

Load the three MGF libraries — the Spectraverse v1.0.1 deposit from Zenodo (doi:10.5281/zenodo.17870921), MassSpecGym.mgf from HuggingFace, and the NPLIB1 archive from Zenodo doi:10.5281/zenodo.8316682 — and parse the SMILES structure annotations from the spectrum metadata. Standardize the SMILES and collapse all three libraries into a single union table of unique chemical structures, keeping a per-library presence flag for each structure. Compute a 2D UMAP embedding of the structures with umap-learn using n_neighbors=5000 and min_dist=0.5, parameters chosen to favor global structure so that between-library coverage differences are visible. Render a 2D kernel-density plot per library over the shared embedding (saved as umap_density_2d.png) so overlap and unique regions are visible, and export the x/y UMAP coordinates for every unique structure with library-membership annotations to a CSV. The comparison is qualitative: the expected result is that Spectraverse's ~44,237 unique small molecules afford visibly broader chemical-space coverage than MassSpecGym or NPLIB1.

## Related tools

- **umap-learn** (Computes the 2D UMAP embedding of unique structures with n_neighbors=5000 and min_dist=0.5)
- **Python** (Implementation language for parsing MGF metadata, standardizing SMILES, embedding, and plotting (Python 3.10+)) — https://github.com/skinniderlab/spectraverse-analysis
- **matchms** (MGF parsing and spectrum metadata handling for the spectral libraries being compared) — https://github.com/matchms/matchms

## Examples

```
import umap; emb = umap.UMAP(n_neighbors=5000, min_dist=0.5).fit_transform(fingerprint_matrix)
```

## Evaluation signals

- umap_density_2d.png exists and shows kernel-density shading for all three libraries over a shared UMAP embedding.
- The exported CSV contains exactly one row per unique structure in the cross-library union, with non-null x/y coordinates and library-membership annotations.
- UMAP parameters match the protocol: n_neighbors=5000, min_dist=0.5.
- The density plot qualitatively reproduces the paper's finding that Spectraverse covers broader chemical space than MassSpecGym and NPLIB1.
- Per-library presence flags in the CSV are consistent with the counts of unique SMILES parsed from each source MGF.

## Limitations

- The coverage comparison is a qualitative 2D density visualization, not a quantitative chemical-space metric.
- Near-duplicate handling matters: near-duplicate spectra removal was only explicitly reported for MassSpecGym comparisons, so overlap between libraries may be inflated by duplicates.
- Requires that SMILES annotations in all three MGF files be standardizable into a common representation before the union is built.

## Evidence

- [abstract] Spectraverse affords the most comprehensive coverage of chemical space of any machine-learning-ready library of MS/MS spectra to date: "The resulting resource affords the most comprehensive coverage of chemical space of any machine-learning-ready library of MS/MS spectra to date"
- [methods] MGF input format: "Prepare your data: Place raw data files (MGF format) in the `data/` directory."
- [intro] Spectraverse library size: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small"
- [other] UMAP workflow parameters: [evidence span withheld — anchor status: not_found]
- [other] Union construction step: [evidence span withheld — anchor status: not_found]
