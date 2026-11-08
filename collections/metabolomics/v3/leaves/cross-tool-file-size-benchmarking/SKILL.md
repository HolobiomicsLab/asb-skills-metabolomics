---
name: cross-tool-file-size-benchmarking
description: 'Use when choosing between two visualization tools for large ordination
  datasets: generate equivalent outputs from both on the same samples and metadata,
  then measure and compare total output file size on disk.'
when_to_use_negative:
- The dataset is small or the output size is not a deciding factor — the 1.3% ratio
  argument matters mainly for thousands of samples and many metadata columns
- The candidate tools do not produce comparable outputs (e.g., one produces interactive
  3D plots and the other a numeric summary), so the size ratio would not be a fair
  comparison
- You need publication-quality static figures rather than web-shareable interactive
  visualizations, which is the trade-off this benchmark quantifies
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Produces the lightweight interactive 3D PCoA visualizations (HTML5/WebGL,
    command line interface) whose file size is benchmarked
  repo: http://emperor.colorado.edu
- name: KiNG
  role: Generates the baseline static per-metadata-field 3D plots whose 1.85 GB output
    size is used as the comparison reference
- name: QIIME
  role: Pipeline through which KiNG is invoked and whose principal coordinates and
    metadata mapping files feed both tools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:2a14b6c5c591c014406ce8ecfafbcd022bbad0d955f631cb6417261d4a1abf8f
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
n_claims: 9
---

# cross-tool-file-size-benchmarking

## Summary

Benchmarks the on-disk output size of competing visualization tools (e.g., EMPeror vs. KiNG) on an identical PCoA dataset and metadata mapping, quantifying the ratio to justify adoption of a lightweight tool. In the source study, EMPeror files were 26 MB versus 1.85 GB for KiNG (1.3% of original size).

## When to use

Use when choosing between two visualization tools for large ordination datasets: generate equivalent outputs from both on the same samples and metadata, then measure and compare total output file size on disk.

## When NOT to use

- The dataset is small or the output size is not a deciding factor — the 1.3% ratio argument matters mainly for thousands of samples and many metadata columns
- The candidate tools do not produce comparable outputs (e.g., one produces interactive 3D plots and the other a numeric summary), so the size ratio would not be a fair comparison
- You need publication-quality static figures rather than web-shareable interactive visualizations, which is the trade-off this benchmark quantifies

## Inputs

- QIIME principal coordinates files (PCoA) for the combined dataset of references [3, 20-22] (5,740 samples)
- QIIME metadata mapping files (120 columns of metadata)
- Example files and supporting datasets from the GigaScience Database deposit

## Outputs

- EMPeror interactive 3D visualization files (discrete and gradient coloring modes), ~26 MB total
- KiNG static per-metadata-field 3D plot files via the QIIME pipeline (discrete and gradient coloring), ~1.85 GB total
- file_size_comparison.csv containing measured sizes and the EMPeror-to-KiNG size ratio

## How to apply

Assemble the same dataset for both tools: QIIME principal coordinates files and metadata mapping files (here 5,740 sample spheres, 120 metadata columns). Run the EMPeror command line interface to emit interactive 3D visualizations in both discrete and gradient coloring modes, then run KiNG via the QIIME pipeline to generate the equivalent static per-metadata-field files for both coloring modes. Measure the total size on disk of each tool's outputs and compute the ratio of EMPeror size to KiNG size. Record measured sizes and ratio in file_size_comparison.csv. Validation: the ratio should be approximately 1.3% (26 MB vs 1.85 GB), matching the paper's reported finding; a result far above this indicates the tools were not run on equivalent outputs or coloring modes.

## Related tools

- **EMPeror** (Produces the lightweight interactive 3D PCoA visualizations (HTML5/WebGL, command line interface) whose file size is benchmarked) — http://emperor.colorado.edu
- **KiNG** (Generates the baseline static per-metadata-field 3D plots whose 1.85 GB output size is used as the comparison reference)
- **QIIME** (Pipeline through which KiNG is invoked and whose principal coordinates and metadata mapping files feed both tools)

## Evaluation signals

- file_size_comparison.csv exists and contains the measured sizes for both tools
- EMPeror output totals approximately 26 MB for the 5,740-sample, 120-metadata-column dataset
- KiNG output totals approximately 1.85 GB for the same dataset and coloring modes
- Computed EMPeror-to-KiNG size ratio is approximately 1.3%, matching the reported finding
- Both discrete and gradient coloring modes were generated for each tool before measuring

## Limitations

- EMPeror command line interface runs on OS X (10.6 and higher) and Linux only; no Windows support is stated
- EMPeror depends on Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent; the GUI requires the Chrome browser
- No version number for EMPeror, no code repository URL (only http://emperor.colorado.edu), and no archival DOI or accession IDs for the supporting datasets are given

## Evidence

- [abstract] File-size ratio finding: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [discussion] Data availability: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [abstract] Load-time scaling rationale: "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
