---
name: biplot-taxa-overlay-in-ordination-space
description: Use when you have a QIIME principal coordinates file plus a metadata
  mapping file and need to display taxa (biplot arrows) or compare two ordinations
  (procrustes) in the same interactive 3D scatter.
when_to_use_negative:
- You need generic 3D plotting without microbiome metadata integration — commercial
  or generic packages such as SpotFire, MATLAB, Mathematica, R, Excel or Matplotlib
  are the alternatives the article contrasts with EMPeror
- 'The command line interface environment is unsupported: EMPeror''s CLI is limited
  to OS X (10.6 and higher) and Linux only, with Python 2.7, QIIME (python libraries
  only), NumPy, BIOM 1.1.0 and PyCogent as requirements'
- The visualization must render in a browser other than Chrome — the stated requirement
  is Chrome specifically, so compatibility elsewhere is untested
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Renders the 3D PCoA scatter and overlays the biplot/procrustes auxiliary-data
    layers as an interactive WebGL web page
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream framework that generates the principal coordinates file and metadata
    mapping file consumed by EMPeror for the base plot and biplot overlay
- name: PyCogent
  role: Listed as a Python library dependency of the EMPeror command line interface
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:78f0ddfa45cc0798e39cd0321c6e6baf93be892e9a6c311214b7e6ba8ba63afa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - PyCogent
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# biplot-taxa-overlay-in-ordination-space

## Summary

Overlay taxon biplot arrows and auxiliary layers (procrustes comparisons, jackknifed beta diversity plots) onto a 3D principal coordinates visualization using EMPeror, so intrinsic data patterns in the ordination space can be interpreted alongside the sample points.

## When to use

Use when you have a QIIME principal coordinates file plus a metadata mapping file and need to display taxa (biplot arrows) or compare two ordinations (procrustes) in the same interactive 3D scatter.

## When NOT to use

- You need generic 3D plotting without microbiome metadata integration — commercial or generic packages such as SpotFire, MATLAB, Mathematica, R, Excel or Matplotlib are the alternatives the article contrasts with EMPeror
- The command line interface environment is unsupported: EMPeror's CLI is limited to OS X (10.6 and higher) and Linux only, with Python 2.7, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent as requirements
- The visualization must render in a browser other than Chrome — the stated requirement is Chrome specifically, so compatibility elsewhere is untested

## Inputs

- QIIME principal coordinates file (3D PCoA ordinaiton of beta diversity distance matrices)
- metadata mapping file
- taxon biplot data (taxa as environmental samples to overlay as arrows)
- second principal coordinates file for procrustes coordinate comparisons

## Outputs

- Standalone interactive HTML5/WebGL web page showing the 3D sample scatter with overlaid biplot arrows
- Procrustes layer connecting corresponding samples across two ordinations in the same plot
- Optional jackknifed beta diversity plot layer for statistical confidence in clustering

## How to apply

Run the EMPeror command line interface on the QIIME principal coordinates file and metadata mapping file to build the base interactive 3D scatter of microbial community samples (PCoA is the dimensionality-reduction step upstream). Then overlay the taxon biplot data as auxiliary elements (taxa arrows) in the same ordination space; for procrustes analysis, supply a second coordinate file so corresponding samples from the two ordinations are compared and connected in the plot. Export the result as a standalone web page and verify it renders in Chrome with both the sample spheres and the auxiliary layers visible and interactive. The rationale is EMPeror's auxiliary-data display mechanism: the small WebGL/HTML5 output file (e.g. 26 MB vs 1.85 GB in KiNG for 5,740 samples) embeds taxa and coordinate comparisons without extra plugins.

## Related tools

- **EMPeror** (Renders the 3D PCoA scatter and overlays the biplot/procrustes auxiliary-data layers as an interactive WebGL web page) — http://emperor.colorado.edu
- **QIIME** (Upstream framework that generates the principal coordinates file and metadata mapping file consumed by EMPeror for the base plot and biplot overlay)
- **PyCogent** (Listed as a Python library dependency of the EMPeror command line interface)

## Evaluation signals

- The exported HTML output opens in Chrome and renders the sample spheres of the 3D principal coordinates plot
- Biplot taxa arrows are visible in the same ordination space as the sample points
- Procrustes layer is present: corresponding samples from the two ordinations are compared/connected in the plot
- Auxiliary layers are interactive (dynamic metadata-based recoloring/visibility) rather than static images
- Output file size stays small enough for embedding in emails or web pages without extra plugins (e.g. 26 MB scale rather than the ~1.85 GB KiNG files for 5,740 samples)

## Limitations

- Command line interface is OS X (10.6 and higher) and Linux only; no Windows support stated
- Requires Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent — an end-of-life Python version and a single-browser GUI dependency
- No version number or code repository URL stated for Emperor; only the project home page http://emperor.colorado.edu is given

## Evidence

- [abstract] ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates, show taxa as well as environmental samples,: "ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates, show taxa as well as environmental samples,"
- [discussion] interactively manipulating these datasets to add auxiliary data and visualization techniques: "and interactively manipulating these datasets to add auxiliary data and visualization techniques"
- [abstract] display jackknifed-resampled data to assess statistical confidence in clustering: "display jackknifed-resampled data to assess statistical confidence in clustering"
