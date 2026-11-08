---
name: qiime-principal-coordinates-input-processing
description: Use when input data are QIIME principal coordinates files plus metadata
  mapping files (e.g., 5,740 samples and 120 metadata columns) that must be coupled
  for interactive 3D ordination with discrete or gradient coloring.
when_to_use_negative:
- You only have beta diversity distance matrices — principal coordinates analysis
  (PCoA) must be run first to produce the principal coordinates files this skill consumes.
- Your execution environment is Windows-only — EMPeror's command line interface is
  supported on OS X (10.6 and higher) and Linux only.
- Your stack cannot supply the stated requirements (Python 2.7, QIIME python libraries,
  NumPy, BIOM 1.1.0, PyCogent) — the end-of-life Python 2.7 dependency makes this
  the wrong choice for a modern Python 3 environment.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3941
tools:
- name: EMPeror
  role: Consumes the coupled PCoA coordinates and metadata mapping via its versatile
    command line interface to produce interactive 3D WebGL visualizations with discrete
    and gradient coloring
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Produces the principal coordinates files and mapping files consumed by this
    skill and supplies the python-library dependency; also the pipeline through which
    baseline KiNG plots are run
- name: KiNG
  role: Baseline 3D plot generator run via QIIME for the file-size comparison (1.85
    GB of static per-metadata-field files vs EMPeror's 26 MB)
- name: PyCogent
  role: Required Python dependency of the EMPeror environment alongside Python 2.7,
    NumPy and BIOM 1.1.0
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:fa62fc08ef6aa38ba11a97349ad803417f094b9538da5c5ace4762bb0d29217b
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# qiime-principal-coordinates-input-processing

## Summary

Couple QIIME principal coordinates files with metadata mapping files into the input EMPeror needs to render interactive 3D PCoA visualizations of microbial community data, replacing KiNG's per-metadata-field static files that balloon to gigabyte scale.

## When to use

Use when input data are QIIME principal coordinates files plus metadata mapping files (e.g., 5,740 samples and 120 metadata columns) that must be coupled for interactive 3D ordination with discrete or gradient coloring.

## When NOT to use

- You only have beta diversity distance matrices — principal coordinates analysis (PCoA) must be run first to produce the principal coordinates files this skill consumes.
- Your execution environment is Windows-only — EMPeror's command line interface is supported on OS X (10.6 and higher) and Linux only.
- Your stack cannot supply the stated requirements (Python 2.7, QIIME python libraries, NumPy, BIOM 1.1.0, PyCogent) — the end-of-life Python 2.7 dependency makes this the wrong choice for a modern Python 3 environment.

## Inputs

- QIIME principal coordinates files (PCoA coordinate matrices derived from beta diversity distance matrices, one row per sample/sphere)
- QIIME metadata mapping files (sample-by-column metadata tables; 120 columns in the combined dataset)
- Example/supporting datasets from the GigaScience Database deposit or the EMPeror FTP site (combined 5,740-sample dataset)

## Outputs

- EMPeror interactive 3D visualization bundle (HTML5/WebGL/JavaScript) supporting discrete and gradient coloring
- Shareable output files totaling ~26 MB for the 5,740-sample, 120-metadata-column dataset, embeddable in emails or web pages without extra plugins
- file_size_comparison.csv recording EMPeror vs KiNG output sizes and the ~1.3% ratio (when reproducing the benchmark)

## How to apply

Obtain the QIIME principal coordinates files and metadata mapping files from the GigaScience Database deposit or the EMPeror FTP site; each PCoA row is one sample (sphere) derived from principal coordinates analysis of beta diversity distance matrices, and the mapping file carries the metadata columns (120 in the combined dataset of references [3,20-22]). Parse and join both files on sample identifiers so every sphere can be recolored or hidden dynamically as a function of metadata, in both discrete (categorical) and gradient (continuous) modes. Feed the coupled tables to EMPeror's command line interface, which writes an HTML5/WebGL/JavaScript visualization loadable in Chrome; the environment requires Python 2.7, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent, and the CLI runs on OS X (10.6+) and Linux only. Prefer this route over running KiNG via the QIIME pipeline to emit static per-metadata-field files: for 5,740 samples x 120 columns KiNG output totals 1.85 GB while EMPeror totals 26 MB (1.3%), with load times in seconds instead of many minutes. When benchmarking, measure on-disk sizes of both output sets and compute the EMPeror-to-KiNG ratio into a table such as file_size_comparison.csv, expecting approximately 1.3%.

## Related tools

- **EMPeror** (Consumes the coupled PCoA coordinates and metadata mapping via its versatile command line interface to produce interactive 3D WebGL visualizations with discrete and gradient coloring) — http://emperor.colorado.edu
- **QIIME** (Produces the principal coordinates files and mapping files consumed by this skill and supplies the python-library dependency; also the pipeline through which baseline KiNG plots are run)
- **KiNG** (Baseline 3D plot generator run via QIIME for the file-size comparison (1.85 GB of static per-metadata-field files vs EMPeror's 26 MB))
- **PyCogent** (Required Python dependency of the EMPeror environment alongside Python 2.7, NumPy and BIOM 1.1.0)

## Evaluation signals

- file_size_comparison.csv exists and records EMPeror output of approximately 26 MB versus 1.85 GB for KiNG on the combined 5,740-sample, 120-metadata-column dataset (ratio approximately 1.3%).
- Every sample/sphere in the principal coordinates file resolves to a metadata row in the mapping file, so both discrete (categorical) and gradient (continuous) recoloring work without dropped points.
- The generated visualization renders thousands of points with minimal load times (seconds rather than the many minutes KiNG needs) and opens in Chrome without extra plugins.
- The interactive UI supports modifying perceptible plot aspects — different principal coordinates axes, metadata-driven recoloring and point visibility — as required for rapid exploratory investigation.

## Limitations

- Command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support is stated.
- Depends on Python 2.7 (an end-of-life Python version), Chrome (GUI appears tested in Chrome only), QIIME python libraries, NumPy, BIOM 1.1.0 and PyCogent.
- Supporting example files and data sets carry no archival DOI or accession IDs — only the GigaScience Database and EMPeror FTP site are cited as sources.

## Evidence

- [abstract] PCoA inputs for dimensionality reduction: "Ordination methods, such as Principal Coordinates Analysis (PCoA) [6] are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices"
- [abstract] File-size benchmark vs KiNG: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] Load-time scaling expectation: "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [discussion] Environment requirements: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [discussion] Where to obtain the input datasets: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] OS support constraint: "OS X (10.6 and higher) and Linux only for the command line interface."
