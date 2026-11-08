---
name: lcms-processing-software-benchmarking
description: Use when comparing the computational efficiency (CPU time, memory, scaling
  with sample number) of LC-MS metabolomics processing tools on centroided mzML datasets,
  or when validating claims that a new tool scales better than established ones like
  XCMS.
when_to_use_negative:
- Input data are low-resolution MS or non-Orbitrap platforms, since asari's settings
  are optimized for high-resolution Orbitrap data and benchmarking feature quality
  there would not be representative
- The goal is feature detection accuracy/quantification comparison rather than computational
  resource benchmarking
- Data are profile-mode or non-mzML vendor formats without prior centroiding/conversion
  (e.g., via ThermoRawFileParser or msConvert)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: asari
  role: Tool under benchmark; processes centroided mzML files (`asari process --mode
    pos --input <dir>`), timing measured in a fresh process per subset
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Baseline comparison tool; peak calling and grouping steps run on identical
    mzML subsets with documented default parameters
- name: Python
  role: Used for asari execution and resource-measurement wrapper (resource module
    / GNU time) capturing CPU time, wall-clock time, and peak memory
- name: ThermoRawFileParser
  role: Used to convert Thermo .RAW files to .mzML when preparing centroided input
    data for the benchmark
  repo: https://github.com/compomics/ThermoRawFileParser
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:92f88b94ff30c9e7e82a9f5c96d06417813b9c793c671de338280642d705d132
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - ThermoRawFileParser
  - XCMS
  - asari
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lcms-processing-software-benchmarking

## Summary

Benchmark the computational performance of LC-MS metabolomics data processing tools (e.g., asari vs XCMS) by measuring CPU time, wall-clock time, and peak memory on dataset subsets of increasing sample size, then fitting scaling curves to quantify performance gaps and scalability.

## When to use

Use when comparing the computational efficiency (CPU time, memory, scaling with sample number) of LC-MS metabolomics processing tools on centroided mzML datasets, or when validating claims that a new tool scales better than established ones like XCMS.

## When NOT to use

- Input data are low-resolution MS or non-Orbitrap platforms, since asari's settings are optimized for high-resolution Orbitrap data and benchmarking feature quality there would not be representative
- The goal is feature detection accuracy/quantification comparison rather than computational resource benchmarking
- Data are profile-mode or non-mzML vendor formats without prior centroiding/conversion (e.g., via ThermoRawFileParser or msConvert)

## Inputs

- Public centroided mzML metabolomics dataset (subsets of 5, 10, 20, 50, 100 files; e.g., SLAW dataset subsets)
- asari installed via pip3 (asari-metabolomics, Python 3.8+)
- XCMS installed from R/Bioconductor

## Outputs

- Tabulated CPU time, wall-clock time, and peak memory per tool per sample size
- Scaling ratios (runtime growth vs sample count)
- Fitted scaling curves comparing asari vs XCMS quantifying the performance gap and linearity of scaling

## How to apply

Select subsets of increasing sample size (e.g., 5, 10, 20, 50, 100 files) from a public centroided mzML metabolomics dataset to systematically vary study size. Run asari (`pip3 install asari-metabolomics`; `asari process --mode pos --input <dir>`) on each subset in a fresh process, capturing user CPU time, wall-clock time, and peak memory via a resource-measurement wrapper such as the Python resource module or GNU time. Run XCMS (R/Bioconductor, peak calling and grouping steps) on the identical subsets with documented default parameters, capturing the same three resource metrics. Tabulate all measurements per tool per sample size, compute scaling ratios (time vs sample count), and fit/plot scaling curves to characterize growth rate. Accept the benchmark if the newer tool shows substantially lower CPU time (approximately 1-2 orders of magnitude for asari vs XCMS) and its runtime grows approximately linearly with sample number, consistent with the paper's reported performance figures.

## Related tools

- **asari** (Tool under benchmark; processes centroided mzML files (`asari process --mode pos --input <dir>`), timing measured in a fresh process per subset) — https://github.com/shuzhao-li/asari
- **XCMS** (Baseline comparison tool; peak calling and grouping steps run on identical mzML subsets with documented default parameters)
- **Python** (Used for asari execution and resource-measurement wrapper (resource module / GNU time) capturing CPU time, wall-clock time, and peak memory)
- **ThermoRawFileParser** (Used to convert Thermo .RAW files to .mzML when preparing centroided input data for the benchmark) — https://github.com/compomics/ThermoRawFileParser

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- asari shows approximately 1-2 orders-of-magnitude lower CPU time than XCMS on the same subsets
- asari CPU time and memory use scale largely linearly with sample number on SLAW dataset subsets
- Same three metrics (user CPU time, wall-clock time, peak memory) captured consistently for both tools across all sample sizes
- Each asari run executed in a fresh process so resource measurements are not contaminated by prior runs
- Scaling ratios computed per tool per sample size confirm the growth-rate characterization before fitting curves

## Limitations

- The software does not work with low resolution data
- Asari has mostly been tested on Orbitrap platforms; settings need further optimization for other platforms
- Efficiency is not optimal on data from longer chromatography
- Data of very high noise may need customized processing
- MS/MS spectra are ignored by asari in the default LC-MS workflow

## Evidence

- [other] Benchmarking asari against XCMS on datasets of varying sizes showed a 1-2 orders-of-magnitude improvement in CPU time for asari, and CPU time and memory use scaled largely linearly with sample number on SLAW dataset subsets.: [evidence span withheld — anchor status: not_found]
- [abstract] It offers substantial improvement in computational performance over current tools, and it is highly scalable: "It offers substantial improvement in computational performance over current tools, and it is highly scalable"
- [readme] Asari command usage from README: "To process all mzML files under directory mydir/projectx_dir: `asari process --mode pos --input mydir/projectx_dir`"
- [readme] README on asari scalability design: "Scalable, performance conscious, disciplined use of memory and CPU"
- [discussion] Discussion on platform limitations: "Asari has a short history and has been mostly tested on the Orbitrap platforms. The settings will need further optimization for other platforms."
