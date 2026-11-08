---
name: browser-rendering-timing-measurement
description: Use when comparing load/render performance of browser-based 3D ordination
  viewers (e.g. EMPeror vs KiNG) on large PCoA datasets with thousands of samples
  and rich metadata mapping files.
when_to_use_negative:
- The goal is to interactively explore or recolor an ordination plot rather than to
  benchmark loading performance of visualization tools
- The dataset is small enough (few samples/metadata columns) that both tools load
  quickly and a timing comparison is uninformative
- The comparison targets statistical accuracy of the ordination (e.g. jackknifed beta
  diversity or procrustes analysis) rather than file-size and load-time scaling
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Browser-based (Chrome) 3D visualization tool whose output file size and load
    time are measured; generated via its command line interface
  repo: http://emperor.colorado.edu
- name: KiNG
  role: Baseline 3D plot renderer whose output file size (1.85 GB) and load time (many
    minutes) provide the comparison reference
- name: QIIME
  role: Generates the KiNG-format 3D plot files from principal coordinates and mapping
    files via its KiNG export path
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:c492484e12136324616e7bb63a3657248cfef1b5fab623a516e0137559720ceb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - QIIME
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# browser-rendering-timing-measurement

## Summary

A benchmarking method that times how fast browser-based 3D visualization tools load large PCoA ordination datasets, comparing EMPeror's web-gl visualization against QIIME's KiNG export path on a 5,740-sample, 120-metadata-column microbial community dataset.

## When to use

Use when comparing load/render performance of browser-based 3D ordination viewers (e.g. EMPeror vs KiNG) on large PCoA datasets with thousands of samples and rich metadata mapping files.

## When NOT to use

- The goal is to interactively explore or recolor an ordination plot rather than to benchmark loading performance of visualization tools
- The dataset is small enough (few samples/metadata columns) that both tools load quickly and a timing comparison is uninformative
- The comparison targets statistical accuracy of the ordination (e.g. jackknifed beta diversity or procrustes analysis) rather than file-size and load-time scaling

## Inputs

- QIIME principal coordinates file (PCoA results) for the 5,740-sample combined dataset
- QIIME metadata mapping file with 120 metadata columns
- KiNG-format 3D plot files generated via QIIME's KiNG export path (discrete and gradient coloring variants)
- EMPeror visualization output generated from the same principal coordinates and mapping files via the EMPeror command line interface

## Outputs

- load_time_comparison.csv containing measured browser load times and file sizes for KiNG and EMPeror across discrete and gradient coloring variants
- Computed ratio of EMPeror to KiNG file size (expected ~1.3%, i.e. 26 MB vs 1.85 GB)

## How to apply

Download the deposited example files (QIIME principal coordinates file and metadata mapping file for the combined 5,740-sample, 120-column dataset) from the GigaScience Database and the EMPeror FTP site. Generate KiNG-format 3D plot files for both the discrete and gradient coloring variants using QIIME's KiNG export path, which replicates the coordinates for each metadata category, and generate the EMPeror visualization from the same principal coordinates and mapping files via EMPeror's command line interface. Time the browser loading of both outputs in Chrome (the GUI browser specified in EMPeror's requirements), repeating the measurement for the discrete and gradient coloring variants. Record the file sizes of the KiNG outputs (1.85 GB for both variants) and the EMPeror output (26 MB) alongside the measured load times, write all timings and sizes to load_time_comparison.csv, and compute the ratio of EMPeror to KiNG file size. Accept the result if the ratio is approximately 1.3% (26 MB vs 1.85 GB) and EMPeror loads in seconds while KiNG takes many minutes, matching the paper's reported performance claims.

## Related tools

- **EMPeror** (Browser-based (Chrome) 3D visualization tool whose output file size and load time are measured; generated via its command line interface) — http://emperor.colorado.edu
- **KiNG** (Baseline 3D plot renderer whose output file size (1.85 GB) and load time (many minutes) provide the comparison reference)
- **QIIME** (Generates the KiNG-format 3D plot files from principal coordinates and mapping files via its KiNG export path)

## Evaluation signals

- EMPeror output file size is approximately 1.3% of the KiNG file size (26 MB vs 1.85 GB)
- EMPeror visualization loads in seconds in Chrome while KiNG files take many minutes to load
- Both the discrete and gradient coloring variants are timed, with repeated measurements recorded
- load_time_comparison.csv contains both load times and file sizes for all variants
- File-size ratio is computed from the recorded sizes, not estimated

## Limitations

- EMPeror's GUI requires the Chrome browser, suggesting untested compatibility with other browsers
- EMPeror's command line interface supports only OS X (10.6 and higher) and Linux; no Windows support is stated
- EMPeror depends on Python 2.7, NumPy, BIOM 1.1.0, PyCogent and QIIME (python libraries only); no version number is stated for Emperor itself
- The supporting example files are available via the GigaScience Database and EMPeror FTP site without archival DOIs or accession IDs

## Evidence

- [abstract] In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] scale to thousands of points with minimal load times (seconds versus many minutes in KiNG): "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [discussion] The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25].: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
