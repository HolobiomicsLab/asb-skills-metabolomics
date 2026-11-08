---
name: pcoa-plot-reproduction-from-deposited-files
description: Use when you have a deposited QIIME principal coordinates file and mapping
  file (e.g. the 5,740-sample, 120-column combined dataset) and need to rebuild a
  PCoA 3D plot or benchmark visualization loading performance.
when_to_use_negative:
- You do not have the original deposited principal coordinates file or mapping file
  — the skill regenerates plots from these exact artifacts, not from raw sequence
  data.
- You need statistical analysis of the ordination itself rather than plot reproduction
  or loading benchmarking.
- Your environment lacks Chrome or cannot run the EMPeror Python 2.7 stack (OS X 10.6+
  or Linux only for the command line interface).
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0622
tools:
- name: EMPeror
  role: Generates the web-browser PCoA visualization from the principal coordinates
    and mapping files via its command line interface, producing the compact output
    used in the load-time comparison.
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Produces the principal coordinates file from beta diversity distance matrices
    and provides the KiNG export path that replicates coordinates per metadata category
    for the baseline comparison.
- name: KiNG
  role: Loads the legacy QIIME-generated 3D plot files whose load times and 1.85 GB
    file sizes serve as the benchmark against EMPeror.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:394f2c3754a07fc7169910c6ec7bda35e21af9ab3a8a7ebc6775e31c822b32f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - QIIME
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# pcoa-plot-reproduction-from-deposited-files

## Summary

Reproduce published PCoA visualizations by regenerating EMPeror plots from the deposited QIIME principal coordinates and metadata mapping files, comparing load-time and file-size performance against the legacy KiNG QIIME export path.

## When to use

Use when you have a deposited QIIME principal coordinates file and mapping file (e.g. the 5,740-sample, 120-column combined dataset) and need to rebuild a PCoA 3D plot or benchmark visualization loading performance.

## When NOT to use

- You do not have the original deposited principal coordinates file or mapping file — the skill regenerates plots from these exact artifacts, not from raw sequence data.
- You need statistical analysis of the ordination itself rather than plot reproduction or loading benchmarking.
- Your environment lacks Chrome or cannot run the EMPeror Python 2.7 stack (OS X 10.6+ or Linux only for the command line interface).

## Inputs

- QIIME principal coordinates file (PCoA coordinates for the combined 5,740-sample dataset)
- QIIME metadata mapping file (~120 metadata columns)
- EMPeror CLI (Python 2.7 with QIIME libraries, NumPy, BIOM 1.1.0, PyCogent)
- KiNG for loading the legacy 3D plot files
- Chrome browser for timing visualization load

## Outputs

- load_time_comparison.csv with per-variant (discrete/gradient) load times and file sizes
- EMPeror-to-KiNG file size ratio (~1.3%, 26 MB vs 1.85 GB)
- Reproducible EMPeror visualization of the PCoA ordination colored by metadata

## How to apply

Download the example files from the GigaScience Database or the EMPeror FTP site, then generate the visualization two ways: (1) export KiNG-format 3D plot files via QIIME's KiNG export path, which replicates the coordinates for each metadata category for both discrete and gradient coloring, and (2) run the EMPeror command line interface on the same principal coordinates and mapping files. Load both outputs in Chrome (EMPeror's stated browser requirement), timing load for discrete and gradient variants and recording file sizes; compute the EMPeror/KiNG size ratio. Accept the result when EMPeror's output is roughly 1.3% of the KiNG size (26 MB vs 1.85 GB for 5,740 samples and 120 metadata columns) and loads in seconds while KiNG takes many minutes.

## Related tools

- **EMPeror** (Generates the web-browser PCoA visualization from the principal coordinates and mapping files via its command line interface, producing the compact output used in the load-time comparison.) — http://emperor.colorado.edu
- **QIIME** (Produces the principal coordinates file from beta diversity distance matrices and provides the KiNG export path that replicates coordinates per metadata category for the baseline comparison.)
- **KiNG** (Loads the legacy QIIME-generated 3D plot files whose load times and 1.85 GB file sizes serve as the benchmark against EMPeror.)

## Examples

```
python script/make_emperor.py -i pcoa.txt -m mapping.txt  # then open the generated visualization in Chrome and time the load
```

## Evaluation signals

- EMPeror output file size is approximately 1.3% of the KiNG output size (26 MB vs 1.85 GB for the 5,740-sample dataset).
- EMPeror visualization loads in seconds while KiNG files take many minutes to load in Chrome.
- Both discrete and gradient coloring variants are tested, with timings and sizes logged for each.
- The regenerated EMPeror plot reproduces the published ordination pattern from the deposited coordinates and mapping file.
- load_time_comparison.csv contains one row per variant with file size, load time, and tool.

## Limitations

- EMPeror's command line interface runs only on OS X (10.6 and higher) and Linux; no Windows support is stated.
- Requires Python 2.7 (end-of-life), Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent — the GUI needs Chrome specifically, suggesting untested compatibility with other browsers.
- Load-time measurements are browser-dependent and were reported as seconds vs many minutes rather than exact figures; timing methodology must be repeated consistently.
- No archival DOI or accession IDs are given for the supporting example files; retrieval depends on the GigaScience Database and EMPeror FTP site links remaining available.

## Evidence

- [abstract] 1.3% of KiNG file size for 5,740 samples: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [discussion] example files deposited: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [abstract] load-time claim: "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [discussion] EMPeror requirements: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
