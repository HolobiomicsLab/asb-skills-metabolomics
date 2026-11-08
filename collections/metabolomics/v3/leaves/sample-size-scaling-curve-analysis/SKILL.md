---
name: sample-size-scaling-curve-analysis
description: Use when you need to quantify how a data-processing tool's CPU time and
  memory scale with sample number, or to benchmark one metabolomics pipeline against
  another (e.g. asari vs XCMS) on identical subsets of a public dataset.
when_to_use_negative:
- The input is already a processed feature table — scaling analysis requires running
  the tools end-to-end on raw centroided mzML files
- The question is feature-detection accuracy or annotation quality rather than computational
  performance; this skill measures only CPU time, wall time, and memory
- The data are low-resolution or from platforms asari was not tuned for, since absolute
  runtimes would not generalize (asari was mostly tested on Orbitrap platforms)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_0091
tools:
- name: asari
  role: 'Tool under benchmark: runs LC-MS metabolomics processing on each sample-size
    subset while CPU time, wall-clock time, and peak memory are captured'
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: 'Comparison baseline: peak calling and grouping steps run with documented
    default parameters on the identical subsets'
- name: Python
  role: Used for asari execution (Python 3.8+) and for wrapping runs with the resource
    module to capture CPU time and peak memory
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:d2bf0a18bdfd7ec66372aec6ba35eb439ba67ea4078197da58a5832960e17837
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# sample-size-scaling-curve-analysis

## Summary

A benchmarking method that varies the number of input samples (e.g. 5, 10, 20, 50, 100 centroided mzML files) and measures CPU time, wall-clock time, and peak memory for competing tools to quantify absolute performance gaps and scaling behavior. In the source work it was used to show asari's 1-2 orders-of-magnitude CPU-time improvement over XCMS and its near-linear scaling.

## When to use

Use when you need to quantify how a data-processing tool's CPU time and memory scale with sample number, or to benchmark one metabolomics pipeline against another (e.g. asari vs XCMS) on identical subsets of a public dataset.

## When NOT to use

- The input is already a processed feature table — scaling analysis requires running the tools end-to-end on raw centroided mzML files
- The question is feature-detection accuracy or annotation quality rather than computational performance; this skill measures only CPU time, wall time, and memory
- The data are low-resolution or from platforms asari was not tuned for, since absolute runtimes would not generalize (asari was mostly tested on Orbitrap platforms)

## Inputs

- Subsets of increasing sample count (e.g. 5, 10, 20, 50, 100 files) extracted from a public centroided mzML metabolomics dataset (e.g. SLAW dataset subsets)
- Competing processing tools installed in isolated environments (asari via pip3 install asari-metabolomics; XCMS via R/Bioconductor)
- Resource-measurement wrapper (Python resource module or GNU time) for user CPU time, wall-clock time, and peak memory

## Outputs

- Table of CPU time, wall-clock time, and peak memory per tool per sample size
- Scaling ratios (time vs sample count) characterizing the growth rate
- Fitted/plot scaling curves comparing tools (e.g. asari vs XCMS) and a quantified performance gap

## How to apply

Select nested subsets of increasing sample size (e.g. 5, 10, 20, 50, 100 files) from a public centroided mzML dataset such as SLAW to systematically vary study size. Run each tool — e.g. `asari process --mode pos --input <dir>` and XCMS's peak calling and grouping steps with documented default parameters — on the identical subsets in fresh processes, capturing user CPU time, wall-clock time, and peak memory via a resource-measurement wrapper (Python resource module or GNU time). Tabulate all measurements per tool per sample size and compute scaling ratios (time vs sample count) to characterize the growth rate. Fit and plot scaling curves to verify near-linear growth; the source study accepted the benchmark when asari showed substantially lower CPU time (~1-2 orders of magnitude) and approximately linear scaling with sample number, consistent with reported figures.

## Related tools

- **asari** (Tool under benchmark: runs LC-MS metabolomics processing on each sample-size subset while CPU time, wall-clock time, and peak memory are captured) — https://github.com/shuzhao-li/asari
- **XCMS** (Comparison baseline: peak calling and grouping steps run with documented default parameters on the identical subsets)
- **Python** (Used for asari execution (Python 3.8+) and for wrapping runs with the resource module to capture CPU time and peak memory)

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- asari's CPU time on the same subsets is 1-2 orders of magnitude lower than XCMS's, matching the paper's reported performance gap
- Fitted CPU-time vs sample-count curves for asari are approximately linear (linear scaling with sample number), as reported for SLAW dataset subsets
- Memory use also scales approximately linearly with sample number for asari
- All three metrics (user CPU time, wall-clock time, peak memory) are captured for every tool-sample-size combination, with each tool run in a fresh process on identical subsets
- Tool versions, parameters (e.g. asari --mode pos; XCMS default parameters), and dataset subset composition are documented for reproducibility

## Limitations

- Absolute runtimes are specific to the tested platforms and data: asari was mostly tested on Orbitrap platforms, does not work with low-resolution data, and its efficiency is not optimal on longer chromatography data
- The benchmark measures only computational resources; feature-detection quality differences between tools are assessed separately
- Very high noise data may need customized processing, which could shift runtimes away from the benchmarked regime

## Evidence

- [results] Benchmarking asari against XCMS on datasets of varying sizes showed a 1-2 orders-of-magnitude improvement in CPU time for asari, and CPU time and memory use scaled largely linearly with sample number on SLAW dataset subsets.: [evidence span withheld — anchor status: not_found]
- [intro] Asari offers substantial improvement of computational performance over the previous tools, and is highly scalable.: "Asari offers substantial improvement of computational performance over the previous tools, and is highly scalable."
- [abstract] It offers substantial improvement in computational performance over current tools, and it is highly scalable: "It offers substantial improvement in computational performance over current tools, and it is highly scalable"
- [readme] To process all mzML files under directory mydir/projectx_dir: asari process --mode pos --input mydir/projectx_dir: "To process all mzML files under directory mydir/projectx_dir:

`asari process --mode pos --input mydir/projectx_dir`"
- [discussion] Asari has a short history and has been mostly tested on the Orbitrap platforms.: "Asari has a short history and has been mostly tested on the Orbitrap platforms."
