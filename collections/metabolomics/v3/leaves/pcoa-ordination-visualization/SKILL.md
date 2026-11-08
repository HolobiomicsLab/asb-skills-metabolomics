---
name: pcoa-ordination-visualization
description: Use when you have a QIIME principal coordinates file and a metadata mapping
  file (e.g. thousands of samples, many metadata columns) and need an exploratory
  3D ordination visualization colored by sample metadata.
when_to_use_negative:
- You need upstream analysis (e.g. generating the beta diversity distance matrix or
  the PCoA coordinates themselves) — QIIME handles the upstream analysis and EMPeror
  only visualizes the resulting coordinates.
- The dataset is small and a generic 3D plotting package (R, MATLAB, Matplotlib, SpotFire,
  KiNG) already meets your needs without metadata-integrated recoloring of thousands
  of points.
- You require a Windows-supported CLI or a browser other than Chrome — the EMPeror
  command line interface is stated as OS X (10.6+) and Linux only, and the GUI requires
  Chrome.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Generates the interactive WebGL 3D PCoA visualization via its command line
    interface and provides the gradient/discrete metadata coloring controllers used
    to recolor samples
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Produces the principal coordinates file (from beta diversity distance matrices)
    and provides the python libraries that EMPeror depends on; its mapping file supplies
    the metadata for coloring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:1972aa592878abe2f6a18417ed096340980e936e625bc53c23e421e60cf08d55
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pcoa-ordination-visualization

## Summary

Visualize high-throughput microbial community beta diversity as an interactive 3D PCoA plot with EMPeror, recoloring points dynamically as a function of metadata gradients (e.g. age) or discrete categories so that patterns such as infant-versus-adult separation become directly viewable.

## When to use

Use when you have a QIIME principal coordinates file and a metadata mapping file (e.g. thousands of samples, many metadata columns) and need an exploratory 3D ordination visualization colored by sample metadata.

## When NOT to use

- You need upstream analysis (e.g. generating the beta diversity distance matrix or the PCoA coordinates themselves) — QIIME handles the upstream analysis and EMPeror only visualizes the resulting coordinates.
- The dataset is small and a generic 3D plotting package (R, MATLAB, Matplotlib, SpotFire, KiNG) already meets your needs without metadata-integrated recoloring of thousands of points.
- You require a Windows-supported CLI or a browser other than Chrome — the EMPeror command line interface is stated as OS X (10.6+) and Linux only, and the GUI requires Chrome.

## Inputs

- QIIME principal coordinates file (output of PCoA on a beta diversity distance matrix)
- QIIME metadata mapping file with the metadata columns to color by (e.g. age as gradient or category)
- Optional: QIIME biom table / jackknifed beta diversity results for advanced plot types

## Outputs

- Interactive WebGL 3D PCoA visualization (EMPeror HTML/Javascript output, small enough to embed in emails or web pages)
- Exported PNG rendering of the recolored ordination plot (e.g. figure_1b_age_coloring.png)

## How to apply

Start from a principal coordinates analysis (PCoA) of a beta diversity distance matrix to reduce the data to a low-dimensional coordinate file. Download the example principal coordinates and mapping files (e.g. the combined 5,740-sample / 120-metadata-column dataset) from the GigaScience Database or EMPeror FTP site, then run the EMPeror command line interface on the principal coordinates and mapping files to generate an interactive WebGL/HTML5 visualization, opening it in Chrome. Use EMPeror's controllers to apply gradient coloring, so each point's color is a function of its position in a metadata gradient (e.g. age), or discrete/categorical coloring for categorical fields, and dynamically change point visibility or switch principal coordinate axes. Export a rendering of the relevant comparison (e.g. infant time-series samples versus healthy adult samples) as a PNG. Rationale: EMPeror's metadata coloring and small output size (26 MB vs 1.85 GB in KiNG for 5,740 samples and 120 metadata columns) make exploratory analysis of large-scale microbiome ordinations fast and shareable.

## Related tools

- **EMPeror** (Generates the interactive WebGL 3D PCoA visualization via its command line interface and provides the gradient/discrete metadata coloring controllers used to recolor samples) — http://emperor.colorado.edu
- **QIIME** (Produces the principal coordinates file (from beta diversity distance matrices) and provides the python libraries that EMPeror depends on; its mapping file supplies the metadata for coloring)

## Evaluation signals

- The exported plot reproduces the reported result: integrating age metadata into the coloring clearly shows the age differences between infant samples and samples from healthy human adults (expert visual comparison against published Figure 1B panels).
- The output PNG is loadable/valid.
- Both coloring variants are applied: gradient coloring (color as a function of position in the age metadata gradient) and discrete/categorical coloring.
- The visualization loads in seconds with thousands of points (e.g. 5,740 samples), reflecting EMPeror's small file size (26 MB vs 1.85 GB in KiNG for the same data).
- Every sample point's color corresponds to its value in the chosen metadata column of the mapping file.

## Limitations

- The command line interface is limited to OS X (10.6 and higher) and Linux; no Windows support is stated.
- Requires Python 2.7 (end-of-life), Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent; the GUI is only validated in Chrome.
- No version number for EMPeror itself and no code repository URL are provided in the article; only the project home page is given.
- No archival DOI or accession IDs are given for the supporting example files and datasets, which must be fetched from the GigaScience Database or the EMPeror FTP site.

## Evidence

- [abstract] EMPeror command line interface generates the WebGL visualization from PCoA + mapping files: "Here we present EMPeror, an open source and web browser enabled tool with a versatile command line interface that allows researchers to perform rapid exploratory investigations of 3D visualizations"
- [abstract] PCoA is the dimensionality-reduction step feeding the visualization: "Ordination methods, such as Principal Coordinates Analysis (PCoA) [6] are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices"
- [abstract] Metadata coloring is EMPeror's core capability for this skill: "EMPeror includes a rich set of controllers to modify features as a function of the metadata. By being specifically tailored to the requirements of microbial ecologists"
- [abstract] Scale/memory advantage vs KiNG for the 5,740-sample dataset: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [discussion] Where the example input files are obtained: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Dependency and browser constraints: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
