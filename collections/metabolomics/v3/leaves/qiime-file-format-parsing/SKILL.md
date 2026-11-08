---
name: qiime-file-format-parsing
description: Use when the input consists of QIIME-format principal coordinates files
  (including jackknifed replicate PCoA files) and a metadata mapping file that must
  be converted into per-sample coordinates and category memberships for 3D visualization
  or recoloring/show/hide controllers.
when_to_use_negative:
- The data are not in QIIME principal-coordinates/mapping-file formats (e.g. arbitrary
  CSVs or feature tables), since the parsers target those specific QIIME file types
- The goal is upstream sequencing or distance-matrix computation rather than preparing
  ordination data for 3D visualization
- The visualization output already exists and you only need to interact with the EMPeror
  page in Chrome
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QIIME
  role: Provides the python libraries used to parse its principal coordinates files
    and metadata mapping files before visualization
- name: PyCogent
  role: Python library used together with QIIME libraries for parsing the QIIME PCoA
    and mapping files
- name: EMPeror
  role: Consumer of the parsed data; its Python command line interface accepts QIIME
    principal coordinates files and metadata mapping files and emits the standalone
    HTML5/WebGL page
  repo: http://emperor.colorado.edu
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:6455980c3fcf31e71ff36c0604f5bdac675b0dcf0cad57de5d695f210e4bfc8f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - PyCogent
  - QIIME
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# qiime-file-format-parsing

## Summary

Parse QIIME principal coordinates files and metadata mapping files into per-sample 3D PCoA coordinates and per-sample metadata category memberships using the QIIME/PyCogent Python libraries, so the data can drive EMPeror's metadata-based visualization controls.

## When to use

Use when the input consists of QIIME-format principal coordinates files (including jackknifed replicate PCoA files) and a metadata mapping file that must be converted into per-sample coordinates and category memberships for 3D visualization or recoloring/show/hide controllers.

## When NOT to use

- The data are not in QIIME principal-coordinates/mapping-file formats (e.g. arbitrary CSVs or feature tables), since the parsers target those specific QIIME file types
- The goal is upstream sequencing or distance-matrix computation rather than preparing ordination data for 3D visualization
- The visualization output already exists and you only need to interact with the EMPeror page in Chrome

## Inputs

- QIIME principal coordinates file (PCoA of a beta diversity distance matrix)
- QIIME metadata mapping file (per-sample metadata columns, including MIxS-family fields)
- Jackknifed beta diversity replicate principal coordinates files (optional)

## Outputs

- Per-sample 3D coordinates keyed by sample identifier
- Per-sample metadata category memberships for every mapping-file column
- Parsed coordinate plus metadata payload embedded in the generated EMPeror HTML5/WebGL visualization page

## How to apply

Read the QIIME principal coordinates file to extract per-sample 3D coordinates and the metadata mapping file to extract per-sample columns; the article's workflow uses the QIIME python libraries and PyCogent under Python 2.7 alongside NumPy and BIOM 1.1.0. For jackknifed beta diversity inputs, parse all replicate PCoA files so per-replicate coordinates can be aggregated into per-sample confidence regions. Keep the parsed sample identifiers keyed so coordinates, metadata categories, and any auxiliary data (biplots, procrustes coordinate comparisons) stay aligned per sample. For the paper's combined example (5,740 samples, 120 metadata columns), embed the parsed coordinates and the full set of metadata columns directly into the generated EMPeror HTML5/WebGL page rather than emitting static per-category files, which is what reduces file size to 26 MB versus 1.85 GB for KiNG. Validate by loading the resulting page in Chrome and confirming per-category recolor, show, and hide controllers respond interactively to the selected metadata category.

## Related tools

- **QIIME** (Provides the python libraries used to parse its principal coordinates files and metadata mapping files before visualization)
- **PyCogent** (Python library used together with QIIME libraries for parsing the QIIME PCoA and mapping files)
- **EMPeror** (Consumer of the parsed data; its Python command line interface accepts QIIME principal coordinates files and metadata mapping files and emits the standalone HTML5/WebGL page) — http://emperor.colorado.edu

## Evaluation signals

- Parsing the paper's combined example yields 5,740 samples and 120 metadata columns with all sample identifiers matched between the PCoA file and the mapping file
- Per-sample coordinates extracted from the principal coordinates file reproduce the original PCoA axis values exactly
- Every metadata column (including MIxS-family fields) is enumerated in the generated control panel
- The generated page loads in Chrome in seconds rather than minutes and stays near the reported 26 MB (1.3% of the KiNG equivalent)
- Selecting different metadata categories visibly recolors and shows/hides the correct spheres for that category

## Limitations

- Requires Python 2.7, an end-of-life Python version, plus QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent
- The command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support is stated
- The graphical user interface requires Chrome, suggesting untested compatibility with other browsers
- Example files and supporting data sets are available in the GigaScience Database and EMPeror FTP site, but no archival DOI or accession IDs are given

## Evidence

- [abstract] QIIME parsing requirement: "QIIME [7], an open source framework for upstream and downstream analysis of microbial community samples generated via high-throughput sequencing instruments, typically generates 3D plots using KiNG"
- [discussion] Dependency list for parsing stack: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [abstract] Ordination input rationale: "Ordination methods, such as Principal Coordinates Analysis (PCoA) [6] are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices"
- [discussion] Where example input files come from: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
