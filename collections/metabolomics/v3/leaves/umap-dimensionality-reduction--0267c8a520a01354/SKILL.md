---
name: umap-dimensionality-reduction
description: Use when you have SMILES structure annotations from two or more MGF-format
  MS/MS spectral libraries and want to visualize and compare their chemical-space
  coverage and overlap via dimensionality reduction.
when_to_use_negative:
- You need a quantitative, statistically rigorous chemical-space comparison (e.g.,
  descriptor-space diversity metrics) rather than a qualitative 2D visualization
- Input spectra lack SMILES structure annotations, so no molecular feature matrix
  can be built from the libraries
- You are comparing spectra-level properties (peaks, adducts, ionization modes) rather
  than structure-level chemical-space coverage
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0121
tools:
- name: Python
  role: Implementation language for SMILES standardization, the UMAP embedding, and
    density plotting (Python 3.10 or higher per the article's environment setup)
  repo: https://github.com/skinniderlab/spectraverse-analysis
- name: umap-learn
  role: Computes the 2D UMAP embedding of unique structures with n_neighbors=5000
    and min_dist=0.5
- name: matchms
  role: Used in the surrounding pipeline for importing and cleaning the MGF-format
    MS/MS libraries whose SMILES annotations feed this visualization
  repo: https://github.com/matchms/matchms
- name: pip
  role: Package manager used to install the analysis environment (pip install -e .)
  repo: https://github.com/skinniderlab/spectraverse-analysis
- name: conda
  role: Creates the spectraverse analysis environment from environment.yml
  repo: https://github.com/skinniderlab/spectraverse-analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.5c06256
    title: Comprehensive Curation and Harmonization of Small-Molecule MS/MS Libraries
      in Spectraverse
  source_content_hash: sha256:8a5afe10a268e455f29e3a42c91ae1aaea46c8a093fc7ff5c241dfd1528b5fe7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# umap-dimensionality-reduction

## Summary

Compute a 2D UMAP embedding of standardized, deduplicated chemical structures (SMILES) pooled from Spectraverse, MassSpecGym, and NPLIB1, then render per-library 2D density shading to qualitatively compare chemical-space coverage across MS/MS spectral libraries.

## When to use

Use when you have SMILES structure annotations from two or more MGF-format MS/MS spectral libraries and want to visualize and compare their chemical-space coverage and overlap via dimensionality reduction.

## When NOT to use

- You need a quantitative, statistically rigorous chemical-space comparison (e.g., descriptor-space diversity metrics) rather than a qualitative 2D visualization
- Input spectra lack SMILES structure annotations, so no molecular feature matrix can be built from the libraries
- You are comparing spectra-level properties (peaks, adducts, ionization modes) rather than structure-level chemical-space coverage

## Inputs

- Spectraverse MGF library (Zenodo doi:10.5281/zenodo.17870921) with SMILES metadata
- MassSpecGym.mgf from HuggingFace with SMILES metadata
- NPLIB1 archive (Zenodo doi:10.5281/zenodo.8316682) with SMILES metadata
- Union table of standardized, unique SMILES with per-library presence flags

## Outputs

- umap_density_2d.png — 2D density plot over the UMAP embedding, one kernel-density shading per library
- CSV coordinate table of 2D UMAP x/y coordinates for every unique structure, annotated with library membership

## How to apply

Parse the SMILES annotations from the spectrum metadata of each MGF library: Spectraverse (Zenodo doi:10.5281/zenodo.17870921, 488,630 spectra / 44,237 unique small molecules), MassSpecGym.mgf (HuggingFace), and NPLIB1 (Zenodo doi:10.5281/zenodo.8316682). Standardize the SMILES and collapse them into a single union table of unique structures, keeping a per-library presence flag for each structure. Compute a 2D UMAP embedding of the structures with umap-learn using n_neighbors=5000 and min_dist=0.5, which forces broad global structure so that library-scale differences dominate the layout. Render a kernel-density-shaded 2D plot with one density layer per library over the shared embedding so coverage and overlap can be compared directly, and export the x/y coordinates for every unique structure with library-membership annotations to CSV.

## Related tools

- **Python** (Implementation language for SMILES standardization, the UMAP embedding, and density plotting (Python 3.10 or higher per the article's environment setup)) — https://github.com/skinniderlab/spectraverse-analysis
- **umap-learn** (Computes the 2D UMAP embedding of unique structures with n_neighbors=5000 and min_dist=0.5)
- **matchms** (Used in the surrounding pipeline for importing and cleaning the MGF-format MS/MS libraries whose SMILES annotations feed this visualization) — https://github.com/matchms/matchms
- **pip** (Package manager used to install the analysis environment (pip install -e .)) — https://github.com/skinniderlab/spectraverse-analysis
- **conda** (Creates the spectraverse analysis environment from environment.yml) — https://github.com/skinniderlab/spectraverse-analysis

## Examples

```
import umap; coords = umap.UMAP(n_neighbors=5000, min_dist=0.5, n_components=2).fit_transform(fingerprint_matrix)
```

## Evaluation signals

- Both umap_density_2d.png and the CSV coordinate table exist after the run
- The CSV has exactly one row per unique structure in the SMILES union across all three libraries, with x and y coordinates populated (no NaNs)
- Each row carries per-library membership flags consistent with the standardized SMILES deduplication (a structure present in multiple libraries is a single row with multiple flags)
- The density visualization qualitatively reproduces the paper's reported result that Spectraverse affords broader chemical-space coverage than MassSpecGym and NPLIB1
- Embedded parameters are reproducible: n_neighbors=5000 and min_dist=0.5 recorded alongside the output

## Limitations

- The coverage comparison is qualitative (2D density shading); UMAP is stochastic and sensitive to random seed and hyperparameters, so exact layouts vary between runs
- Coverage differences partly reflect library size — Spectraverse contains 488,630 spectra from 44,237 unique small molecules, more than twice as many spectra as MassSpecGym — so the plot shows coverage breadth, not per-compound quality
- The article notes that curation pitfalls may remain beyond what was manually reviewed, so structure annotations feeding the embedding may contain residual incoherent representations

## Evidence

- [abstract] Spectraverse broader coverage claim: "The resulting resource affords the most comprehensive coverage of chemical space of any machine-learning-ready library of MS/MS spectra to date"
- [intro] Library scale: "Spectraverse contains 488,630 MS/MS spectra from 44,237 unique small molecules"
- [intro] Comparison baselines: "Spectraverse expands on the largest existing collection of public
MS/MS spectra that has been used to train and benchmark machine-learning
models, MassSpecGym, by almost 250,000 spectra"
- [readme] Environment setup: "conda env create -f environment.yml; conda activate spectraverse"
- [methods] Input format: "Place raw data files (MGF format) in the `data/` directory."
