---
name: microbiome-metadata-driven-plot-styling
description: Use when you have a QIIME principal coordinates file plus a metadata
  mapping file and need the ordination points colored, toggled, or made transparent
  by metadata categories or gradients in a 3D web visualization.
when_to_use_negative:
- The plotting target is not a PCoA/beta-diversity ordination of microbial community
  samples (e.g. raw feature tables or phylogenetic trees)
- You need a command-line GUI workflow on Windows — the EMPeror command line interface
  supports OS X (10.6 and higher) and Linux only
- You cannot use Chrome — the stated runtime requirements list Chrome explicitly for
  the interactive interface
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3697
tools:
- name: EMPeror
  role: Renders the 3D PCoA visualization with a rich set of controllers that modify
    point features (color, visibility, transparency) as a function of the metadata
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream source of the principal coordinates files and metadata mapping files
    consumed by the EMPeror command line interface
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:5d9c49686506ed624517df06f470f31b8c09ebba1cd0cdd241b6efc225bd7e91
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# microbiome-metadata-driven-plot-styling

## Summary

Style EMPeror 3D PCoA scatter plots (colors, visibility, transparency, point shapes) as a function of sample metadata using QIIME metadata mapping files, so that categorical and gradient metadata drive the visual encoding of ordination points. This is what makes EMPeror plots interpretable for exploratory microbial community analysis.

## When to use

Use when you have a QIIME principal coordinates file plus a metadata mapping file and need the ordination points colored, toggled, or made transparent by metadata categories or gradients in a 3D web visualization.

## When NOT to use

- The plotting target is not a PCoA/beta-diversity ordination of microbial community samples (e.g. raw feature tables or phylogenetic trees)
- You need a command-line GUI workflow on Windows — the EMPeror command line interface supports OS X (10.6 and higher) and Linux only
- You cannot use Chrome — the stated runtime requirements list Chrome explicitly for the interactive interface

## Inputs

- QIIME principal coordinates file (PCoA coordinates of a beta diversity distance matrix)
- QIIME metadata mapping file (sample metadata with categorical and/or gradient columns)

## Outputs

- Self-contained WebGL/HTML5 3D PCoA web page with metadata-driven coloring, visibility, and transparency controllers
- Standalone visualization deliverable shareable via email or web embedding without plugins

## How to apply

Parse the QIIME metadata mapping file alongside the principal coordinates file with the EMPeror command line interface (Python 2.7, QIIME python libraries, NumPy, PyCogent, BIOM 1.1.0), so each sample's PCoA coordinates are linked to its metadata categories. Map metadata onto the plot: use metadata categories to drive coloring and visibility of points, supporting both gradients and discrete categorical data, and use the EMPeror controllers to change features as a function of the metadata. Render the result as a hardware-accelerated WebGL/HTML5 scene whose UI controllers let users interactively toggle visibility, adjust transparency, and recolor points by arbitrary metadata. Deliver the scene as a small self-contained web page (EMPeror's output is ~1.3% of KiNG's file size, e.g. 26 MB vs 1.85 GB for 5,740 samples and 120 metadata columns) that can be embedded in emails or web pages without extra plugins.

## Related tools

- **EMPeror** (Renders the 3D PCoA visualization with a rich set of controllers that modify point features (color, visibility, transparency) as a function of the metadata) — http://emperor.colorado.edu
- **QIIME** (Upstream source of the principal coordinates files and metadata mapping files consumed by the EMPeror command line interface)

## Evaluation signals

- Each sample in the mapping file appears as a point (or confidence region) in the rendered scene, positioned at its PCoA coordinates
- Changing a metadata category in the UI controllers changes point color and/or visibility interactively, for both gradients and discrete categories
- The output opens in Chrome without extra plugins and is small enough to embed (EMPeror file sizes on the order of tens of MB rather than GB for thousands of samples)
- Plot loads in seconds rather than minutes at the scale of thousands of samples (the 5,740-sample benchmark)
- Transparency and visibility toggles operate per metadata category as exposed by the UI controllers

## Limitations

- Command line interface supports OS X (10.6 and higher) and Linux only; no Windows support stated
- Depends on Python 2.7 (end-of-life) with QIIME python libraries, NumPy, BIOM 1.1.0 and PyCogent
- Graphical interface requires Chrome, suggesting untested compatibility with other browsers

## Evidence

- [abstract] EMPeror includes a rich set of controllers to modify features as a function of the metadata.: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [abstract] ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates: "ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates"
- [discussion] it contains several user interface features, enabling straightforward modifications and customization of perceptible aspects in the plot plus the incorporation of statistical techniques: "it contains several user interface features, enabling straightforward modifications and customization of perceptible aspects in the plot plus the incorporation of statistical techniques"
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
