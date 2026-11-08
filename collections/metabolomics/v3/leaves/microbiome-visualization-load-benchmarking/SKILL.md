---
name: microbiome-visualization-load-benchmarking
description: Use when a PCoA/beta-diversity visualization of thousands of microbiome
  samples loads too slowly or produces unwieldy plot files, and you need quantitative
  load-time and file-size evidence to choose between visualization tools such as EMPeror
  and KiNG.
when_to_use_negative:
- The input is not PCoA/ordination data derived from a beta diversity distance matrix
  — EMPeror and KiNG both consume principal coordinates and mapping files, not raw
  feature tables or reads.
- The dataset is small (few dozen samples) with sparse metadata, where coordinate
  duplication per category in KiNG does not inflate files enough for load-time differences
  to be meaningful.
- 'You need a browser-independent GUI: EMPeror''s GUI lists only Chrome as a requirement,
  so the benchmark result does not transfer to other browsers without retesting.'
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: EMPeror
  role: Generates the compact HTML5/WebGL 3D PCoA visualization being benchmarked;
    invoked through its command line interface
  repo: http://emperor.colorado.edu
- name: KiNG
  role: Baseline visualization tool whose QIIME-exported 3D plot files (with coordinates
    replicated per metadata category) provide the load-time and file-size comparison
- name: QIIME
  role: Supplies the KiNG export path and the principal coordinates / metadata mapping
    files consumed by both visualizers
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:f3fdee847b79d3ef3717099a07f1bf9dea4c6b00fdc655e6b5bb0217881b8c83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - QIIME
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# microbiome-visualization-load-benchmarking

## Summary

A benchmarking procedure for comparing browser-based 3D ordination visualization tools by loading time and output file size, validated on EMPeror versus KiNG using a 5,740-sample PCoA dataset with 120 metadata columns.

## When to use

Use when a PCoA/beta-diversity visualization of thousands of microbiome samples loads too slowly or produces unwieldy plot files, and you need quantitative load-time and file-size evidence to choose between visualization tools such as EMPeror and KiNG.

## When NOT to use

- The input is not PCoA/ordination data derived from a beta diversity distance matrix — EMPeror and KiNG both consume principal coordinates and mapping files, not raw feature tables or reads.
- The dataset is small (few dozen samples) with sparse metadata, where coordinate duplication per category in KiNG does not inflate files enough for load-time differences to be meaningful.
- You need a browser-independent GUI: EMPeror's GUI lists only Chrome as a requirement, so the benchmark result does not transfer to other browsers without retesting.

## Inputs

- QIIME principal coordinates file (PCoA results) for the dataset under test
- QIIME metadata mapping file (e.g., 120 metadata columns for 5,740 samples)
- QIIME KiNG-export path for generating KiNG-format 3D plot files
- EMPeror command line interface
- Chrome web browser (stated EMPeror requirement)

## Outputs

- load_time_comparison.csv containing measured load times and file sizes for each tool and coloring variant (discrete and gradient)
- Computed ratio of EMPeror output file size to KiNG output file size (expected ~1.3%, i.e., 26 MB vs 1.85 GB)

## How to apply

Download the deposited QIIME principal coordinates file and metadata mapping file for the target dataset (here 5,740 samples, 120 metadata columns) from the GigaScience Database and EMPeror FTP site. Generate KiNG-format 3D plot files via QIIME's KiNG export path, which replicates the coordinates for each metadata category, for both discrete and gradient coloring variants; then generate the EMPeror visualization from the same principal coordinates and mapping files via EMPeror's command line interface. Time the browser (Chrome, per EMPeror's stated requirements) loading of each output for both coloring variants, and record file sizes alongside the timings. Compute the ratio of EMPeror to KiNG file size; write all timings and sizes to load_time_comparison.csv. Accept the result if EMPeror's output is approximately 1.3% of the KiNG size (26 MB vs 1.85 GB) and loads in seconds while KiNG takes many minutes.

## Related tools

- **EMPeror** (Generates the compact HTML5/WebGL 3D PCoA visualization being benchmarked; invoked through its command line interface) — http://emperor.colorado.edu
- **KiNG** (Baseline visualization tool whose QIIME-exported 3D plot files (with coordinates replicated per metadata category) provide the load-time and file-size comparison)
- **QIIME** (Supplies the KiNG export path and the principal coordinates / metadata mapping files consumed by both visualizers)

## Evaluation signals

- EMPeror output is approximately 1.3% of the KiNG output file size (26 MB vs 1.85 GB for 5,740 samples and 120 metadata columns).
- EMPeror visualization loads in the browser within seconds, while KiNG files take many minutes to load.
- load_time_comparison.csv contains timing and size rows for both the discrete and gradient coloring variants.
- Both tools were run on the same principal coordinates file and mapping file, so the comparison isolates the visualization format rather than differing input data.

## Limitations

- EMPeror's command line interface is stated to run on OS X (10.6 and higher) and Linux only; no Windows support is stated.
- EMPeror requires Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent, so the benchmark is tied to Chrome and to Python 2.7 dependencies.
- No version number for Emperor itself is stated in the article, complicating reproducibility of the benchmark.
- No archival DOI or accession IDs are given for the supporting example files and data sets; they are referenced only via the GigaScience Database and EMPeror FTP site.

## Evidence

- [abstract] In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] scale to thousands of points with minimal load times (seconds versus many minutes in KiNG): "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [discussion] The example files and additional data sets supporting the results of this article are available in the GigaScience Database: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
