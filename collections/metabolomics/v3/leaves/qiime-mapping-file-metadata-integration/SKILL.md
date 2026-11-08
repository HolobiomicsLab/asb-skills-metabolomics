---
name: qiime-mapping-file-metadata-integration
description: Use when a QIIME-generated principal coordinates file must be explored
  interactively in 3D with sample metadata (mapping file columns) driving coloring,
  visibility, and axes for thousands of microbial community samples.
when_to_use_negative:
- The ordination file has no accompanying metadata mapping file, so no metadata-driven
  coloring, gradients, or visibility controls are possible
- The goal is static publication figure rendering in generic 3D plotting packages
  (MATLAB, R, Matplotlib) rather than interactive, metadata-linked exploration
- The working environment lacks the stated requirements (Python 2.7, Chrome, QIIME
  python libraries, NumPy, BIOM 1.1.0, PyCogent) or runs only Windows for the command
  line interface
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3315
tools:
- name: EMPeror
  role: Consumes the QIIME principal coordinates file plus the metadata mapping file;
    its rich set of controllers modifies plot features (color, visibility, axes) as
    a function of the metadata and exports a standalone interactive web visualization
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream framework that generates the principal coordinates file (from PCoA
    of beta diversity distance matrices) and the metadata mapping file that EMPeror
    ingests; QIIME python libraries are a stated runtime requirement
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:e20f1fb82e43eaf4c9a18e90212bb3122b8ed7bdcc2d2b8863855079f5cf15b2
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# QIIME mapping-file metadata integration

## Summary

Integrates a QIIME metadata mapping file with a principal coordinates (PCoA) file so that EMPeror can color, show/hide, and reshape sample spheres as a function of sample metadata, supporting gradients, categorical coloring, and parallel-coordinates views.

## When to use

Use when a QIIME-generated principal coordinates file must be explored interactively in 3D with sample metadata (mapping file columns) driving coloring, visibility, and axes for thousands of microbial community samples.

## When NOT to use

- The ordination file has no accompanying metadata mapping file, so no metadata-driven coloring, gradients, or visibility controls are possible
- The goal is static publication figure rendering in generic 3D plotting packages (MATLAB, R, Matplotlib) rather than interactive, metadata-linked exploration
- The working environment lacks the stated requirements (Python 2.7, Chrome, QIIME python libraries, NumPy, BIOM 1.1.0, PyCogent) or runs only Windows for the command line interface

## Inputs

- QIIME principal coordinates file (PCoA ordination of a beta diversity distance matrix)
- QIIME metadata mapping file (sample metadata columns, both discrete categories and numeric gradients)

## Outputs

- Standalone interactive 3D HTML5/WebGL scatter plot of samples colored and shaped by mapping-file metadata
- Optional parallel-coordinates representation of the same metadata-linked ordination

## How to apply

Retrieve the QIIME principal coordinates file and its metadata mapping file (for example from the GigaScience Database or EMPeror FTP site). Run the EMPeror command line interface with both files so the mapping-file columns become controllers that modify plot features as a function of the metadata: discrete columns produce categorical coloring, numeric columns produce gradient coloring, and columns can toggle point visibility. Choose which principal coordinates axes to display and optionally present the data as parallel coordinates. Export the result as a standalone web page (HTML5/WebGL) that can be opened in Chrome and shared via email or web pages without extra plugins; metadata-dependent recoloring and visibility changes remain interactive in the exported file.

## Related tools

- **EMPeror** (Consumes the QIIME principal coordinates file plus the metadata mapping file; its rich set of controllers modifies plot features (color, visibility, axes) as a function of the metadata and exports a standalone interactive web visualization) — http://emperor.colorado.edu
- **QIIME** (Upstream framework that generates the principal coordinates file (from PCoA of beta diversity distance matrices) and the metadata mapping file that EMPeror ingests; QIIME python libraries are a stated runtime requirement)

## Evaluation signals

- The exported HTML opens in Chrome and renders sample spheres without extra plugins
- Changing a mapping-file column in the interface dynamically recolors or toggles visibility of points, confirming metadata columns are wired to the plot controllers
- Both discrete (categorical) and gradient (numeric) metadata columns produce distinct coloring schemes in the plot
- File size of the output is small relative to KiNG-style per-metadata-color files (EMPeror reported 26 MB vs 1.85 GB for 5,740 samples and 120 metadata columns, i.e. 1.3%)
- Visualization loads in seconds rather than the many minutes reported for KiNG at thousands of points

## Limitations

- Command line interface limited to OS X (10.6 and higher) and Linux only; no Windows support stated
- Requires Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent
- Graphical user interface requires Chrome, suggesting untested compatibility with other browsers
- No version number stated for the Emperor software itself and no archival DOI or accession IDs given for supporting example files

## Evidence

- [abstract] EMPeror includes a rich set of controllers to modify features as a function of the metadata.: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [abstract] ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates: "ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates"
- [abstract] QIIME [7], an open source framework for upstream and downstream analysis of microbial community samples generated via high-throughput sequencing instruments, typically generates 3D plots using KiNG: "QIIME [7], an open source framework for upstream and downstream analysis of microbial community samples generated via high-throughput sequencing instruments, typically generates 3D plots"
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
