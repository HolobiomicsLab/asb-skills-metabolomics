---
name: emperor-command-line-visualization-generation
description: Use when you have a PCoA ordination (principal coordinates file) from
  beta diversity analysis plus a mapping/metadata file (e.g., 5,740 samples, 120 metadata
  columns) and need interactive 3D metadata-colored visualization.
when_to_use_negative:
- You only have raw sequence data or a feature table and no PCoA ordination — run
  beta diversity/PCoA first, since EMPeror consumes principal coordinates files, not
  raw data
- You need generic 3D plotting outside the microbial-ecology PCoA + mapping-file workflow
  — generic packages such as MATLAB, R, Excel or Matplotlib are the alternatives the
  paper cites
- Your environment lacks the stated requirements (Python 2.7, Chrome, QIIME python
  libraries, NumPy, BIOM 1.1.0, PyCogent), or you need the command line interface
  on Windows — CLI is stated as OS X (10.6+) and Linux only
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: 'Primary tool: CLI converts principal coordinates + mapping files into an
    interactive WebGL 3D visualization with gradient and discrete metadata-coloring
    controllers'
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream framework whose beta diversity/PCoA output (principal coordinates
    file) and metadata mapping file are the skill's inputs; QIIME python libraries
    are a stated requirement
- name: KiNG
  role: Baseline comparison tool for 3D plots; produces 1.85 GB files for the same
    dataset where EMPeror produces 26 MB
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:b16e72732cf72a7657aab8e1525a41c79888e544245647e87f970142b87802e6
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
n_claims: 12
---

# emperor-command-line-visualization-generation

## Summary

Generate interactive WebGL 3D PCoA visualizations from a QIIME principal coordinates file plus a metadata mapping file using EMPeror's command line interface, then apply gradient or discrete metadata-based coloring (e.g., age) for rapid exploratory analysis of microbial community data.

## When to use

Use when you have a PCoA ordination (principal coordinates file) from beta diversity analysis plus a mapping/metadata file (e.g., 5,740 samples, 120 metadata columns) and need interactive 3D metadata-colored visualization.

## When NOT to use

- You only have raw sequence data or a feature table and no PCoA ordination — run beta diversity/PCoA first, since EMPeror consumes principal coordinates files, not raw data
- You need generic 3D plotting outside the microbial-ecology PCoA + mapping-file workflow — generic packages such as MATLAB, R, Excel or Matplotlib are the alternatives the paper cites
- Your environment lacks the stated requirements (Python 2.7, Chrome, QIIME python libraries, NumPy, BIOM 1.1.0, PyCogent), or you need the command line interface on Windows — CLI is stated as OS X (10.6+) and Linux only

## Inputs

- QIIME principal coordinates file (PCoA ordination of a beta diversity distance matrix)
- QIIME metadata mapping file (e.g., the combined 5,740-sample / 120-column dataset from references 3, 20-22, downloaded from the GigaScience Database or EMPeror FTP site)

## Outputs

- Interactive WebGL/HTML5 3D PCoA visualization generated via the EMPeror command line interface
- Metadata-colored rendering exported as a loadable PNG image (e.g., figure_1b_age_coloring.png)

## How to apply

First run principal coordinates analysis on a beta diversity distance matrix to obtain the principal coordinates file; EMPeror consumes this together with a QIIME metadata mapping file. Run EMPeror's command line interface on both files to produce a WebGL/HTML5 interactive 3D visualization, and open the output in Chrome (the GUI is browser-enabled with no extra plugins). In the browser, use EMPeror's controllers to recolor points as a function of metadata: apply the gradient-coloring controller for continuous columns such as age, or the discrete/categorical coloring variant for categorical columns, and adjust visibility of points to separate groups (e.g., infant time-series samples versus healthy adult samples). Export the rendered panel as a PNG (e.g., figure_1b_age_coloring.png) for publication. The rationale is scale: for 5,740 samples with 120 metadata columns EMPeror produces a 26 MB file versus 1.85 GB for KiNG (1.3% of the size) and loads in seconds rather than many minutes, enabling agile exploratory recoloring.

## Related tools

- **EMPeror** (Primary tool: CLI converts principal coordinates + mapping files into an interactive WebGL 3D visualization with gradient and discrete metadata-coloring controllers) — http://emperor.colorado.edu
- **QIIME** (Upstream framework whose beta diversity/PCoA output (principal coordinates file) and metadata mapping file are the skill's inputs; QIIME python libraries are a stated requirement)
- **KiNG** (Baseline comparison tool for 3D plots; produces 1.85 GB files for the same dataset where EMPeror produces 26 MB)

## Evaluation signals

- The exported plot reproduces the reported result: metadata-integrated age coloring clearly shows the separation of infant time-series samples from healthy adult samples, matching published Figure 1B panels (expert visual comparison)
- The visualization output is a loadable PNG file that opens correctly in Chrome as an interactive WebGL scene
- Both coloring variants succeed: a gradient-coloring panel (Figure 1B-1) and a discrete/categorical panel (Figure 1B-2) rendered from the same inputs
- Output file size stays small relative to KiNG (EMPeror ~26 MB for 5,740 samples / 120 metadata columns, i.e., ~1.3% of KiNG's 1.85 GB), and load time is seconds rather than many minutes
- Every sample in the plot is colored consistently with its position in the chosen metadata gradient or category

## Limitations

- The command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support is stated
- Requires Python 2.7 (an end-of-life Python version), Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent; the GUI depends on Chrome specifically
- No version number or code repository URL is stated for Emperor — only the project home page http://emperor.colorado.edu
- No archival DOI or accession IDs are given for the supporting example files and data sets (GigaScience Database / EMPeror FTP site)

## Evidence

- [abstract] EMPeror CLI purpose: "Here we present EMPeror, an open source and web browser enabled tool with a versatile command line interface that allows researchers to perform rapid exploratory investigations of 3D visualizations"
- [abstract] EMPeror vs KiNG size: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [discussion] Example data availability: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Software requirements: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
