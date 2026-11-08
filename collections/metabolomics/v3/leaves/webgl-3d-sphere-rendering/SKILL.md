---
name: webgl-3d-sphere-rendering
description: Use when you have a QIIME principal coordinates file plus a metadata
  mapping file and need an interactive, plugin-free 3D browser visualization whose
  sphere size/transparency can be modified per metadata category.
when_to_use_negative:
- Target browser lacks WebGL hardware acceleration or is not Chrome, since the paper
  lists Chrome among the hard requirements
- The task is 2D plotting or static figure export (e.g. Matplotlib/R/Excel-style plotting)
  rather than interactive 3D exploratory visualization
- Input ordination coordinates come from a format other than QIIME principal coordinates
  files paired with a metadata mapping file and no converter is available
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: EMPeror
  role: Reference implementation of the WebGL sphere renderer with per-category size/transparency
    controllers; skill re-implements its interactive visualization approach
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream source of the principal coordinates file and metadata mapping file
    consumed by the renderer (python libraries only)
- name: PyCogent
  role: Python dependency listed in EMPeror's requirements, used in the Python parsing
    layer of the visualization pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:7a2bcd103dd9a262775ac9da07307f6ddb189651918dc1d2098baf357f9124b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - PyCogent
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# webgl-3d-sphere-rendering

## Summary

Renders PCoA-ordained microbiome samples as interactive 3D spheres in a browser using hardware-accelerated WebGL (HTML5/Javascript/Python, per EMPeror), exposing sphere size and transparency as per-vertex attributes driven by a per-category metadata controller.

## When to use

Use when you have a QIIME principal coordinates file plus a metadata mapping file and need an interactive, plugin-free 3D browser visualization whose sphere size/transparency can be modified per metadata category.

## When NOT to use

- Target browser lacks WebGL hardware acceleration or is not Chrome, since the paper lists Chrome among the hard requirements
- The task is 2D plotting or static figure export (e.g. Matplotlib/R/Excel-style plotting) rather than interactive 3D exploratory visualization
- Input ordination coordinates come from a format other than QIIME principal coordinates files paired with a metadata mapping file and no converter is available

## Inputs

- QIIME principal coordinates file (per-sample 3D PCoA coordinates from beta-diversity distance matrices)
- QIIME metadata mapping file (per-sample category memberships, e.g. body site)

## Outputs

- Self-contained HTML5/WebGL web page rendering each sample as a 3D sphere
- Interactive per-category controller UI that sets sphere size and transparency as a function of the selected metadata category

## How to apply

Parse the QIIME principal coordinates file and the metadata mapping file in Python to extract per-sample 3D coordinates and per-sample category memberships. Render each sample as a sphere on an HTML5 canvas using WebGL hardware acceleration, exposing sphere size and transparency as per-vertex attributes rather than baked-in constants, so they can be updated dynamically. Implement a controller that maps each selected metadata category (e.g. body site, age) to a size value and a transparency value applied to all spheres belonging to that category, mirroring EMPeror's 'rich set of controllers to modify features as a function of the metadata'. Keep the visualization compact and self-contained: the whole point versus KiNG-style per-category exported plots is that a 5,740-sample, 120-metadata-column plot stays around 26 MB (1.3% of KiNG's 1.85 GB) so it can be embedded in emails or web pages without extra plugins. Target Chrome, since the paper lists 'Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent' as requirements, and package the renderer/controller as a web page deliverable independent of the command line tool.

## Related tools

- **EMPeror** (Reference implementation of the WebGL sphere renderer with per-category size/transparency controllers; skill re-implements its interactive visualization approach) — http://emperor.colorado.edu
- **QIIME** (Upstream source of the principal coordinates file and metadata mapping file consumed by the renderer (python libraries only))
- **PyCogent** (Python dependency listed in EMPeror's requirements, used in the Python parsing layer of the visualization pipeline)

## Evaluation signals

- The web page loads headlessly and renders spheres for every sample present in the principal coordinates file
- Expert review in Chrome confirms that selecting different metadata categories (e.g. body site) visibly changes sphere size and transparency on a per-category basis
- Output page size stays small relative to per-category KiNG exports — on the order of 26 MB for 5,740 samples and 120 metadata columns (1.3% of the 1.85 GB KiNG output)
- Visualization loads in seconds (not many minutes) for thousands of points
- Page runs in a browser with no extra plugins installed

## Limitations

- The graphical user interface requires Chrome; compatibility with other browsers is untested
- The command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support is stated
- Dependencies include Python 2.7, an end-of-life Python version, along with QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent
- No version number, code repository URL, or archival accession for the supporting example files is given — only a project home page and the GigaScience Database/EMPeror FTP site

## Evidence

- [abstract] EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python: "EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python"
- [abstract] rich set of controllers to modify features as a function of the metadata: "EMPeror includes a rich set of controllers to modify features as a function of the metadata"
- [abstract] enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins: "ease of sharing, given EMPeror's small output file size, enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins"
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [abstract] in KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB"
