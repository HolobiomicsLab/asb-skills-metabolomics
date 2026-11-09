---
name: output-file-size-comparison-analysis
description: Use when two alternative tools visualize or transform the same large
  dataset and you must quantify scalability by timing load performance and comparing
  output file sizes, e.g. EMPeror (26 MB) versus KiNG (1.85 GB) for PCoA plots.
when_to_use_negative:
- The tools under comparison consume different input files or datasets, so any file-size
  or load-time difference would confound input size with tool efficiency
- The goal is rendering quality or statistical fidelity of visualizations rather than
  load-time scalability and output size on a large sample/metadata-rich dataset
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: 'Tool under benchmark: its HTML5/WebGL browser-based visualization is generated
    from the PCoA and mapping files via the command line interface, and its output
    file size (26 MB) and Chrome load time are measured'
  repo: http://emperor.colorado.edu
- name: KiNG
  role: 'Baseline tool under benchmark: 3D plots are generated via QIIME''s KiNG export
    path, replicating coordinates per metadata category, and the resulting 1.85 GB
    files'' load times are timed'
- name: QIIME
  role: Provides the KiNG export path that generates the baseline 3D plot files from
    the principal coordinates and metadata mapping files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:ba1cb065ccd140433fb7392c5e7f9dcd066e2a0bed1efeeeac8646d08ac55cba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - QIIME
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# output-file-size-comparison-analysis

## Summary

A benchmarking method that compares competing visualization pipelines by measuring output file size and browser load time on the same dataset, as done for EMPeror versus KiNG on a 5,740-sample PCoA dataset with 120 metadata columns.

## When to use

Use when two alternative tools visualize or transform the same large dataset and you must quantify scalability by timing load performance and comparing output file sizes, e.g. EMPeror (26 MB) versus KiNG (1.85 GB) for PCoA plots.

## When NOT to use

- The tools under comparison consume different input files or datasets, so any file-size or load-time difference would confound input size with tool efficiency
- The goal is rendering quality or statistical fidelity of visualizations rather than load-time scalability and output size on a large sample/metadata-rich dataset

## Inputs

- QIIME principal coordinates file (PCoA results for the 5,740-sample dataset)
- QIIME metadata mapping file with 120 columns
- KiNG-format 3D plot files for discrete and gradient coloring variants (generated via QIIME's KiNG export path)
- EMPeror visualization output generated from the same PCoA and mapping files

## Outputs

- load_time_comparison.csv containing measured browser load times and file sizes for KiNG and EMPeror outputs (discrete and gradient variants)
- Computed ratio of EMPeror to KiNG file size (expected ~1.3%, i.e. 26 MB vs 1.85 GB)

## How to apply

Start from a single shared input set — here a QIIME principal coordinates file plus a metadata mapping file for the combined 5,740-sample, 120-column dataset, downloaded from the GigaScience Database or the EMPeror FTP site. Generate the competing outputs from identical inputs: export KiNG-format 3D plot files through QIIME's KiNG export path (which replicates coordinates for each metadata category, producing discrete and gradient coloring variants) and generate the EMPeror visualization from the same principal coordinates and mapping files via EMPeror's command line interface. Load each output in the browser specified by the tool's requirements (Chrome for EMPeror), timing both the discrete and gradient coloring variants and repeating measurements. Record file sizes alongside measured load times in load_time_comparison.csv and compute the size ratio; accept the comparison if the ratio matches the reported benchmark — EMPeror output approximately 1.3% of KiNG's (26 MB vs 1.85 GB) — and EMPeror loads in seconds while KiNG takes many minutes.

## Related tools

- **EMPeror** (Tool under benchmark: its HTML5/WebGL browser-based visualization is generated from the PCoA and mapping files via the command line interface, and its output file size (26 MB) and Chrome load time are measured) — http://emperor.colorado.edu
- **KiNG** (Baseline tool under benchmark: 3D plots are generated via QIIME's KiNG export path, replicating coordinates per metadata category, and the resulting 1.85 GB files' load times are timed)
- **QIIME** (Provides the KiNG export path that generates the baseline 3D plot files from the principal coordinates and metadata mapping files)

## Evaluation signals

- EMPeror output is approximately 1.3% of the KiNG file size (26 MB vs 1.85 GB) for the 5,740-sample, 120-column dataset
- EMPeror visualization loads in seconds in Chrome while KiNG files load in many minutes
- Both discrete and gradient coloring variants are timed, with measurements repeated rather than taken once
- load_time_comparison.csv contains both file sizes and load times for every variant compared
- KiNG and EMPeror outputs are generated from the identical principal coordinates and metadata mapping files

## Limitations

- EMPeror's graphical user interface requires Chrome, suggesting untested compatibility with other browsers
- EMPeror's command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support is stated
- EMPeror depends on Python 2.7, an end-of-life Python version, plus QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent

## Evidence

- [abstract] KiNG vs EMPeror file size benchmark: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] load-time comparison: "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [discussion] data availability: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] software requirements: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
