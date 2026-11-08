---
name: runtime-memory-profiling-of-batch-pipelines
description: Use when comparing computational performance (CPU time, memory, wall-clock)
  of batch processing pipelines on centroided mzML metabolomics datasets, and when
  verifying claims of scalability with sample number.
when_to_use_negative:
- Input is already a processed feature table rather than raw centroided mzML files
  (no batch peak-processing step remains to profile)
- Data are low-resolution or from long chromatography runs, since asari is stated
  not to work with low-resolution data and its efficiency is not optimal on longer
  chromatography, biasing the runtime comparison
- The goal is measuring feature-detection accuracy or quantification quality rather
  than computational resource usage
edam_topics:
- http://edamontology.org/topic_3892
tools:
- name: asari
  role: Python LC-MS data processing tool whose CPU time, wall-clock time, and peak
    memory are profiled across sample-size subsets; installed via `pip3 install asari-metabolomics`
    and run as `asari process`
  repo: github.com/shuzhao-li-lab/asari
- name: XCMS
  role: R/Bioconductor comparison tool run with documented default peak-calling and
    grouping parameters on identical mzML subsets to establish the performance baseline
- name: Python
  role: Hosts the resource-measurement wrapper (Python resource module) used to capture
    CPU time and memory metrics around each fresh-tool process; asari requires Python
    3.8+
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:b54568ea340fea01d0c375da54c87e33bcbd8761167a79b6681f5c87cc2bac31
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - XCMS
  - asari
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3892
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# runtime-memory-profiling-of-batch-pipelines

## Summary

A benchmarking method that measures user CPU time, wall-clock time, and peak memory of batch data-processing tools (e.g., asari vs XCMS for LC-MS metabolomics mzML files) across increasing sample-size subsets to quantify the performance gap and verify scaling behavior.

## When to use

Use when comparing computational performance (CPU time, memory, wall-clock) of batch processing pipelines on centroided mzML metabolomics datasets, and when verifying claims of scalability with sample number.

## When NOT to use

- Input is already a processed feature table rather than raw centroided mzML files (no batch peak-processing step remains to profile)
- Data are low-resolution or from long chromatography runs, since asari is stated not to work with low-resolution data and its efficiency is not optimal on longer chromatography, biasing the runtime comparison
- The goal is measuring feature-detection accuracy or quantification quality rather than computational resource usage

## Inputs

- Centroided mzML metabolomics data files (LC-MS, e.g., from Thermo .RAW converted with ThermoRawFileParser or msConvert)
- Subsets of increasing sample size (e.g., 5, 10, 20, 50, 100 mzML files) drawn from a public dataset

## Outputs

- Tabulated user CPU time, wall-clock time, and peak memory per tool per sample size
- Scaling ratios of time vs sample count per tool
- Fitted/plotted scaling curves comparing tools (e.g., asari vs XCMS)

## How to apply

Select subsets of increasing sample size (e.g., 5, 10, 20, 50, 100 files) from a public centroided mzML dataset to vary study size systematically. Run each tool (asari via `asari process --mode pos --input <dir>`; XCMS peak calling and grouping in R/Bioconductor with documented defaults) on each subset in a fresh process, wrapping execution with a resource-measurement tool such as the Python resource module or GNU time to capture user CPU time, wall-clock time, and peak memory. Tabulate all measurements per tool per sample size, compute scaling ratios (time vs sample count), then fit and plot scaling curves to compare tools and characterize growth rate. Accept the benchmark when one tool shows substantially lower CPU time (approximately 1-2 orders of magnitude for asari over XCMS) and its runtime grows approximately linearly with sample number.

## Related tools

- **asari** (Python LC-MS data processing tool whose CPU time, wall-clock time, and peak memory are profiled across sample-size subsets; installed via `pip3 install asari-metabolomics` and run as `asari process`) — github.com/shuzhao-li-lab/asari
- **XCMS** (R/Bioconductor comparison tool run with documented default peak-calling and grouping parameters on identical mzML subsets to establish the performance baseline)
- **Python** (Hosts the resource-measurement wrapper (Python resource module) used to capture CPU time and memory metrics around each fresh-tool process; asari requires Python 3.8+)

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- Asari's CPU time on the same mzML subsets is approximately 1-2 orders of magnitude lower than XCMS's
- Asari's CPU time and memory use scale approximately linearly with sample number on the dataset subsets
- All three metrics (user CPU time, wall-clock time, peak memory) are recorded for every tool x sample-size combination, with XCMS run on identical subsets with documented default parameters
- Each tool run is executed in a fresh process so resource measurements are not contaminated by prior state
- Scaling ratios (time vs sample count) computed from the table are consistent with near-linear growth when plotted/fitted

## Limitations

- Asari has mostly been tested on Orbitrap platforms; settings need further optimization for other platforms, which can affect its measured runtime on other instruments
- Asari does not work with low-resolution data and its efficiency is not optimal on data from longer chromatography, so benchmark results may not generalize to those regimes
- MS/MS spectra are ignored by asari in the default LC-MS workflow, so profiling covers the MS1 feature-extraction pipeline only

## Evidence

- [abstract] It offers substantial improvement in computational performance over current tools, and it is highly scalable: "It offers substantial improvement in computational performance over current tools, and it is highly scalable"
- [readme] Input data are centroid mzML files from LC, GC or DI-MS metabolomics.: [evidence span withheld — anchor status: external_doc]
- [readme] Scalable, performance conscious, disciplined use of memory and CPU: "Scalable, performance conscious, disciplined use of memory and CPU"
- [readme] Requires Python 3.8+. Installation time ~ 5 seconds if common libraries already exist.: "Requires Python 3.8+. Installation time ~ 5 seconds if common libraries already exist."
