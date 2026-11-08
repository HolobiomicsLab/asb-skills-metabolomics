---
name: pcoa-plot-visualization
description: Use when you have a QIIME principal coordinates file plus a metadata
  mapping file and need to explore microbial community beta diversity clustering in
  an interactive 3D plot, color or resize points per metadata category, or share the
  visualization in a web browser.
when_to_use_negative:
- You need a generic 2D static scatter of ordination axes only, with no metadata-driven
  interactivity — generic plotting packages such as MATLAB, R, or Matplotlib suffice
- 'You have no metadata mapping file: EMPeror''s per-category controllers and metadata-based
  recoloring/visibility cannot function on coordinates alone'
- Your workflow requires a command line interface on Windows — EMPeror's CLI is OS
  X (10.6 and higher) and Linux only
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Renders the PCoA ordination as an interactive WebGL 3D sphere plot with per-category
    size/transparency/color controllers and dynamic metadata-based recoloring and
    point visibility
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream framework that generates the beta diversity distance matrices, principal
    coordinates file, and metadata mapping file consumed by the visualization
- name: KiNG
  role: Legacy 3D plotting tool for QIIME PCoA output that EMPeror replaces, with
    far smaller file sizes (26 MB vs 1.85 GB) and faster load times
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:be84f0ded6a2e688cd819eaaff46756cbbb616e29eff1a9b5409a05c9bc9dc65
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
n_claims: 13
---

# pcoa-plot-visualization

## Summary

Render Principal Coordinates Analysis (PCoA) ordinations of beta diversity distance matrices as interactive, hardware-accelerated 3D sphere plots using EMPeror's HTML5/WebGL viewer, with per-category controllers for sphere size, transparency, and color driven by QIIME metadata mapping files.

## When to use

Use when you have a QIIME principal coordinates file plus a metadata mapping file and need to explore microbial community beta diversity clustering in an interactive 3D plot, color or resize points per metadata category, or share the visualization in a web browser.

## When NOT to use

- You need a generic 2D static scatter of ordination axes only, with no metadata-driven interactivity — generic plotting packages such as MATLAB, R, or Matplotlib suffice
- You have no metadata mapping file: EMPeror's per-category controllers and metadata-based recoloring/visibility cannot function on coordinates alone
- Your workflow requires a command line interface on Windows — EMPeror's CLI is OS X (10.6 and higher) and Linux only

## Inputs

- QIIME principal coordinates file (per-sample 3D PCoA coordinates from beta diversity distance matrix ordination)
- QIIME metadata mapping file (per-sample category memberships such as body site or age)

## Outputs

- Interactive HTML5/WebGL 3D sphere plot of PCoA samples with per-category size, transparency, and color controllers
- Self-contained web page visualization embeddable in emails or web pages without extra plugins

## How to apply

Run principal coordinates analysis (PCoA) on a beta diversity distance matrix to obtain per-sample 3D coordinates, then supply the QIIME principal coordinates file and metadata mapping file to EMPeror, which renders each sample as a sphere using HTML5, WebGL, Javascript and Python with hardware-accelerated graphics. Expose sphere size and transparency as per-vertex attributes so that the per-category controller can map each selected metadata category (e.g. body site, age) to a size and transparency value applied to all spheres of that category, rather than baking visual properties in as constants. Use EMPeror's rich set of controllers to dynamically recolor points, switch principal coordinates axes, and toggle point visibility as a function of metadata. For statistical context, overlay jackknifed-resampled beta diversity ellipsoids to assess confidence in clustering, and use coordinate comparisons for procrustes analysis plots. Deliver the result as a small self-contained web visualization (EMPeror outputs 26 MB for 5,740 samples and 120 metadata columns versus 1.85 GB in KiNG, i.e. 1.3% of the file size) that can be embedded in emails or web pages without extra plugins.

## Related tools

- **EMPeror** (Renders the PCoA ordination as an interactive WebGL 3D sphere plot with per-category size/transparency/color controllers and dynamic metadata-based recoloring and point visibility) — http://emperor.colorado.edu
- **QIIME** (Upstream framework that generates the beta diversity distance matrices, principal coordinates file, and metadata mapping file consumed by the visualization)
- **KiNG** (Legacy 3D plotting tool for QIIME PCoA output that EMPeror replaces, with far smaller file sizes (26 MB vs 1.85 GB) and faster load times)

## Evaluation signals

- The visualization loads in a browser (Chrome per the stated requirements) in seconds rather than many minutes, even for thousands of points (e.g. 5,740 samples)
- Selecting different metadata categories in the controller visibly changes sphere size and transparency on a per-category basis in the rendered plot
- Output file size is small relative to KiNG-style per-category colored files — expect ~1.3% of the KiNG size for equivalent data (26 MB vs 1.85 GB for 5,740 samples, 120 metadata columns)
- The delivered page is self-contained and runs without extra plugins, so it can be embedded in an email or web page
- Changing the displayed principal coordinates axes and toggling point visibility via the UI correctly reflects the mapping file categories

## Limitations

- Graphical user interface requires Chrome; compatibility with other browsers is not stated
- Command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support stated
- Requires Python 2.7, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent — an end-of-life Python version
- No version number stated for the Emperor software itself and only a project home page URL (no code repository or archival DOI for example files)

## Evidence

- [abstract] EMPeror includes a rich set of controllers to modify features as a function of the metadata.: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [abstract] interactive user interface and hardware-accelerated graphics implemented with HTML5, WebGL, Javascript and Python: "EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python"
- [results] In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [discussion] The example files and additional data sets supporting the results of this article are available in the GigaScience Database: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
