---
name: pcoa-visualization-generation
description: Use when a QIIME principal coordinates file plus a metadata mapping file
  must be turned into an interactive 3D ordination plot with discrete or gradient
  coloring, especially for large datasets (thousands of samples) that must be shared
  or loaded quickly.
when_to_use_negative:
- The task is to generate static per-metadata-field plot files for every metadata
  column — KiNG via QIIME produces these, and EMPeror's single interactive file replaces
  rather than reproduces them.
- The input is not an ordination of a distance matrix (e.g., raw sequence data or
  a feature table) — PCoA dimensionality reduction must be performed upstream before
  visualization.
- 'The working environment is Windows or lacks Chrome: the EMPeror command line interface
  supports only OS X (10.6+) and Linux, and the GUI is specified for Chrome.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: 'Primary tool: renders the PCoA ordination as an interactive 3D WebGL visualization
    with metadata-driven coloring, via its command line interface'
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream framework that generates the principal coordinates files (PCoA of
    beta diversity distance matrices) consumed by EMPeror; only its Python libraries
    are required
- name: KiNG
  role: Baseline/legacy renderer typically used by QIIME for 3D plots; generates large
    static per-metadata-field files used as the size comparison benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:82018a452477b273d6202a221244e14f5175dff2db376b34efbd25b1d35a1967
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pcoa-visualization-generation

## Summary

Generates interactive, browser-based 3D visualizations of Principal Coordinates Analysis (PCoA) ordinations from QIIME principal coordinates files and metadata mapping files using EMPeror, whose WebGL/HTML5 output is orders of magnitude smaller than static per-metadata-field plots from KiNG (26 MB vs 1.85 GB for 5,740 samples).

## When to use

Use when a QIIME principal coordinates file plus a metadata mapping file must be turned into an interactive 3D ordination plot with discrete or gradient coloring, especially for large datasets (thousands of samples) that must be shared or loaded quickly.

## When NOT to use

- The task is to generate static per-metadata-field plot files for every metadata column — KiNG via QIIME produces these, and EMPeror's single interactive file replaces rather than reproduces them.
- The input is not an ordination of a distance matrix (e.g., raw sequence data or a feature table) — PCoA dimensionality reduction must be performed upstream before visualization.
- The working environment is Windows or lacks Chrome: the EMPeror command line interface supports only OS X (10.6+) and Linux, and the GUI is specified for Chrome.

## Inputs

- QIIME principal coordinates file (PCoA ordination of a beta diversity distance matrix)
- QIIME metadata mapping file (with categorical and/or continuous metadata columns)
- Optional taxa/environmental sample overlays or second coordinate sets for biplots and procrustes comparisons

## Outputs

- Interactive HTML5/WebGL 3D EMPeror visualization (~26 MB for 5,740 samples × 120 metadata columns) viewable in Chrome without plugins
- Embeddable visualization suitable for insertion into emails or web pages

## How to apply

Run the PCoA of the beta diversity distance matrix first (e.g., via the QIIME pipeline), then feed the resulting principal coordinates file together with the metadata mapping file to the EMPeror command line interface. Choose discrete coloring for categorical metadata columns or gradient coloring for continuous ones, and let EMPeror emit its compact HTML5/WebGL output that renders in the Chrome browser without extra plugins. For large studies — the paper's benchmark used 5,740 samples (spheres) and 120 metadata columns — prefer EMPeror over KiNG, which would generate a separate static file per metadata field totaling ~1.85 GB versus ~26 MB (1.3%) for EMPeror. Use the controllers to dynamically recolor or toggle visibility of points as a function of metadata, switch principal coordinates axes, and optionally overlay biplots, procrustes coordinate comparisons, or jackknifed-resampled beta diversity plots to assess statistical confidence in clustering.

## Related tools

- **EMPeror** (Primary tool: renders the PCoA ordination as an interactive 3D WebGL visualization with metadata-driven coloring, via its command line interface) — http://emperor.colorado.edu
- **QIIME** (Upstream framework that generates the principal coordinates files (PCoA of beta diversity distance matrices) consumed by EMPeror; only its Python libraries are required)
- **KiNG** (Baseline/legacy renderer typically used by QIIME for 3D plots; generates large static per-metadata-field files used as the size comparison benchmark)

## Evaluation signals

- The EMPeror output loads in a browser (Chrome) with hardware-accelerated graphics and no extra plugins; all 5,740 points render in seconds rather than the many minutes reported for KiNG.
- Total size of EMPeror outputs is on the order of tens of MB (~26 MB) for the 5,740-sample, 120-metadata-column dataset, i.e., ~1.3% of the ~1.85 GB KiNG equivalent.
- Both discrete and gradient coloring modes are produced from the 120 metadata columns, and points can be recolored or hidden dynamically via metadata.
- Different principal coordinates axes can be displayed and switched interactively without regenerating the output file.
- The output file can be embedded in an email or web page and remains shareable given its small size.

## Limitations

- The EMPeror command line interface runs on OS X (10.6 and higher) and Linux only; no Windows support is stated.
- Dependencies are pinned to aging versions: Python 2.7, Chrome, QIIME (Python libraries only), NumPy, BIOM 1.1.0 and PyCogent.
- The graphical user interface requires the Chrome browser; compatibility with other browsers is not claimed.
- No archival DOI or accession IDs are given for the supporting example files, which are available via the GigaScience Database and the EMPeror FTP site.

## Evidence

- [abstract] Ordination methods, such as Principal Coordinates Analysis (PCoA) are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices: "Ordination methods, such as Principal Coordinates Analysis (PCoA) [6] are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices"
- [abstract] EMPeror is an open source web-browser 3D visualization tool with a versatile command line interface: "Here we present EMPeror, an open source and web browser enabled tool with a versatile command line interface that allows researchers to perform rapid exploratory investigations of 3D visualizations"
- [abstract] KiNG output files for discrete and gradient coloring total 1.85 GB vs only 26 MB for EMPeror, i.e., 1.3%: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] EMPeror scales to thousands of points with minimal load times (seconds versus many minutes in KiNG): "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [discussion] EMPeror command line interface supports OS X (10.6 and higher) and Linux only: "OS X (10.6 and higher) and Linux only for the command line interface."
