---
name: pcoa-ordination-plot-generation
description: Use when you have a QIIME principal coordinates (PCoA) file and a metadata
  mapping file and need an interactive, shareable 3D ordination of thousands of microbial
  community samples, optionally overlaid with biplot or procrustes layers.
when_to_use_negative:
- The input is raw high-throughput sequencing data or a beta diversity distance matrix
  that has not yet been ordinated — perform PCoA dimensionality reduction first, then
  plot.
- The goal is a generic 3D plot unrelated to microbial community metadata-driven coloring/controls
  (e.g. a plain scatter from a generic package like MATLAB, R, or Matplotlib may suffice
  instead).
- You require a commercial interactive analytics environment or need a command line
  interface on Windows — the EMPeror CLI is stated to run on OS X (10.6 and higher)
  and Linux only.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: 'primary tool: renders the interactive 3D PCoA scatter with auxiliary-data
    overlays via its command line interface and Chrome WebGL interface'
- name: QIIME
  role: upstream framework generating the principal coordinates file and metadata
    mapping file consumed by EMPeror; its python libraries are a requirement
- name: PyCogent
  role: Python dependency listed among EMPeror requirements, tightly integrated with
    QIIME
- name: KiNG
  role: prior approach for QIIME 3D plots that EMPeror replaces; baseline for file-size
    (1.85 GB vs 26 MB) and load-time comparisons
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:620d7340da5cd09ea92d286bc5bcbaf3c1107ce3fa143cd8df8a6835a151f452
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - PyCogent
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# pcoa-ordination-plot-generation

## Summary

Generates interactive 3D principal coordinates visualizations of microbial community samples from a QIIME PCoA file plus metadata mapping file, with optional auxiliary layers (taxon biplots, procrustes comparisons, jackknifed beta diversity) rendered as a standalone WebGL web page via EMPeror.

## When to use

Use when you have a QIIME principal coordinates (PCoA) file and a metadata mapping file and need an interactive, shareable 3D ordination of thousands of microbial community samples, optionally overlaid with biplot or procrustes layers.

## When NOT to use

- The input is raw high-throughput sequencing data or a beta diversity distance matrix that has not yet been ordinated — perform PCoA dimensionality reduction first, then plot.
- The goal is a generic 3D plot unrelated to microbial community metadata-driven coloring/controls (e.g. a plain scatter from a generic package like MATLAB, R, or Matplotlib may suffice instead).
- You require a commercial interactive analytics environment or need a command line interface on Windows — the EMPeror CLI is stated to run on OS X (10.6 and higher) and Linux only.

## Inputs

- QIIME principal coordinates (PCoA) file
- metadata mapping file
- second coordinates file (for procrustes coordinate comparisons)
- taxon biplot coordinate data (for overlaying taxa as arrows)

## Outputs

- standalone interactive 3D HTML5/WebGL web page rendering sample spheres and any auxiliary layers (biplot arrows, procrustes connections, jackknifed beta diversity plots)
- small output file suitable for embedding in emails or web pages without extra plugins

## How to apply

Run the EMPeror command line interface on the QIIME principal coordinates file and the metadata mapping file to build the base interactive 3D scatter of microbial community samples. The ordination itself derives from principal coordinates analysis (PCoA) of beta diversity distance matrices, used for dimensionality reduction. To add auxiliary data, overlay taxon biplot data as taxa arrows in the same ordination space, or supply a second coordinate file so corresponding samples from two ordinations are compared/connected for procrustes analysis; jackknifed-resampled data can be displayed to assess statistical confidence in clustering. Use the rich set of controllers to recolor or toggle visibility of points as a function of metadata (categorical data or gradients) and to switch principal coordinates axes. Export the result as a standalone web page that must be opened in Chrome, the browser requirement listed for the graphical interface.

## Related tools

- **EMPeror** (primary tool: renders the interactive 3D PCoA scatter with auxiliary-data overlays via its command line interface and Chrome WebGL interface)
- **QIIME** (upstream framework generating the principal coordinates file and metadata mapping file consumed by EMPeror; its python libraries are a requirement)
- **PyCogent** (Python dependency listed among EMPeror requirements, tightly integrated with QIIME)
- **KiNG** (prior approach for QIIME 3D plots that EMPeror replaces; baseline for file-size (1.85 GB vs 26 MB) and load-time comparisons)

## Evaluation signals

- The exported HTML opens in Chrome and renders the sample spheres from the PCoA file without extra plugins.
- Any auxiliary layers added (biplot taxa arrows, procrustes coordinate comparisons) are visible and interactive in the rendered page.
- Recoloring/visibility controllers respond correctly to the columns of the metadata mapping file (categorical and gradient data).
- Output file size stays small relative to legacy KiNG output — the paper reports 26 MB vs 1.85 GB (1.3%) for 5,740 samples and 120 metadata columns.
- Load time for thousands of points is seconds, not the many minutes reported for KiNG.

## Limitations

- Command line interface supports OS X (10.6 and higher) and Linux only; no Windows support stated.
- Requires Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent; the GUI requires Chrome, suggesting untested compatibility with other browsers.
- No version number, code repository URL, or archival DOI/accession IDs are given for the software or the supporting example files.

## Evidence

- [abstract] Ordination methods, such as Principal Coordinates Analysis (PCoA) [6] are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices: "Ordination methods, such as Principal Coordinates Analysis (PCoA) [6] are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices"
- [abstract] EMPeror auxiliary-data display: "ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates, show taxa as well as environmental samples,"
- [abstract] procrustes via coordinate comparisons: "perform coordinate comparisons (useful for procrustes analysis plots)"
- [abstract] In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [discussion] Example files availability: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Requirements: "OS X (10.6 and higher) and Linux only for the command line interface."
