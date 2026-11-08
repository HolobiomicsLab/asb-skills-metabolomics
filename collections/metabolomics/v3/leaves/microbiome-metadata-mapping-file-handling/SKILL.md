---
name: microbiome-metadata-mapping-file-handling
description: Use when you have QIIME principal coordinates files plus a metadata mapping
  file and need to color, recolor, or toggle visibility of PCoA points by metadata
  categories or gradients in an interactive 3D plot.
when_to_use_negative:
- Your working environment is Windows or lacks Python 2.7 with QIIME, NumPy, BIOM
  1.1.0 and PyCogent — the EMPeror command line interface supports only OS X (10.6+)
  and Linux
- You need generic 3D plotting rather than metadata-driven ordination display — tools
  like MATLAB, Mathematica, R, Excel or Matplotlib can always be used instead
- SpotFire-style commercial analytics is affordable and required — EMPeror targets
  open-source, browser-enabled exploratory visualization, not commercial dashboards
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: EMPeror
  role: Consumes the QIIME principal coordinates file and metadata mapping file via
    its command line interface to produce interactive 3D visualizations with discrete
    and gradient coloring
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Supplies the upstream PCoA files and metadata mapping files, and (via its
    python libraries) is a runtime dependency; also runs KiNG for the comparison pipeline
- name: KiNG
  role: Baseline visualizer run through the QIIME pipeline to generate equivalent
    per-metadata-field static plots for file-size and load-time comparison
- name: PyCogent
  role: Listed runtime requirement for EMPeror alongside Python 2.7, NumPy and BIOM
    1.1.0
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:6f4d7a7ee4a2925eae97876a53d5c06053ff64440df575ffabc09ca707167c91
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - PyCogent
  - QIIME
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# microbiome-metadata-mapping-file-handling

## Summary

Prepares QIIME metadata mapping files (sample-by-metadata-category tables, e.g. 120 columns for 5,740 samples) so EMPeror can recolor and filter principal coordinates spheres by discrete categories or gradients, replacing KiNG's per-field static files with one compact interactive 3D visualization.

## When to use

Use when you have QIIME principal coordinates files plus a metadata mapping file and need to color, recolor, or toggle visibility of PCoA points by metadata categories or gradients in an interactive 3D plot.

## When NOT to use

- Your working environment is Windows or lacks Python 2.7 with QIIME, NumPy, BIOM 1.1.0 and PyCogent — the EMPeror command line interface supports only OS X (10.6+) and Linux
- You need generic 3D plotting rather than metadata-driven ordination display — tools like MATLAB, Mathematica, R, Excel or Matplotlib can always be used instead
- SpotFire-style commercial analytics is affordable and required — EMPeror targets open-source, browser-enabled exploratory visualization, not commercial dashboards

## Inputs

- QIIME principal coordinates file (PCoA output from beta diversity distance matrices)
- QIIME metadata mapping file with one column per metadata category (e.g. 120 columns for 5,740 samples)
- Choice of coloring mode per metadata field: discrete (categorical) or gradient (continuous)

## Outputs

- Interactive 3D visualization (HTML5/WebGL, browser-enabled) of PCoA spheres colored by metadata fields
- Optional size-comparison table (file_size_comparison.csv) recording EMPeror output size vs KiNG output size and the ratio

## How to apply

Run PCoA on beta diversity distance matrices to obtain the principal coordinates file, then pair it with the QIIME metadata mapping file whose columns define the coloring categories (the reference case used 120 metadata columns over 5,740 samples). Run the EMPeror command line interface to attach the mapping file to the coordinates and generate an interactive, browser-based (HTML5/WebGL) 3D visualization in which gradients and categorical data drive point color and visibility via its rich set of controllers. Choose discrete coloring for categorical fields and gradient coloring for continuous fields; EMPeror renders both from a single compact output rather than one static file per metadata field. Where a KiNG baseline exists, generate the equivalent QIIME-pipeline KiNG plots for the same fields and compare on-disk sizes: in the published case KiNG totaled 1.85 GB while EMPeror totaled 26 MB (about 1.3%), so use that ratio as a sanity check that the mapping data is being embedded compactly rather than exploded per field. Note the environment constraints: the CLI requires Python 2.7, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent, runs on OS X 10.6+ and Linux only, and the GUI expects Chrome.

## Related tools

- **EMPeror** (Consumes the QIIME principal coordinates file and metadata mapping file via its command line interface to produce interactive 3D visualizations with discrete and gradient coloring) — http://emperor.colorado.edu
- **QIIME** (Supplies the upstream PCoA files and metadata mapping files, and (via its python libraries) is a runtime dependency; also runs KiNG for the comparison pipeline)
- **KiNG** (Baseline visualizer run through the QIIME pipeline to generate equivalent per-metadata-field static plots for file-size and load-time comparison)
- **PyCogent** (Listed runtime requirement for EMPeror alongside Python 2.7, NumPy and BIOM 1.1.0)

## Evaluation signals

- For the combined 5,740-sample, 120-metadata-column dataset, EMPeror outputs total ~26 MB versus 1.85 GB for KiNG, i.e. the EMPeror-to-KiNG ratio is ~1.3% (validate against file_size_comparison.csv)
- Both discrete and gradient coloring modes are produced from the same mapping file, not one output file per metadata field
- The generated visualization loads thousands of points in seconds (vs many minutes in KiNG) and renders in Chrome without extra plugins
- The visualization can be embedded in an email or web page, confirming the output is a self-contained browser file rather than a server-dependent plot
- Recoloring or toggling point visibility by a mapping-file column works interactively via EMPeror's controllers, confirming the metadata were correctly attached to the PCoA sample IDs

## Limitations

- The command line interface runs on OS X (10.6 and higher) and Linux only; no Windows support is stated
- Requires Python 2.7 (end-of-life), Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent, and the GUI is documented only for Chrome
- No version number for EMPeror is stated in the article, and no archival DOI or accession IDs are given for the example files (only GigaScience Database and EMPeror FTP site references)
- Commercial alternatives like SpotFire are out of scope; EMPeror is positioned as the open-source exploratory option rather than an analytics platform

## Evidence

- [abstract] abstract: "Here we present EMPeror, an open source and web browser enabled tool with a versatile command line interface that allows researchers to perform rapid exploratory investigations of 3D visualizations"
- [discussion] discussion: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
