---
name: mzml-dataset-subset-sampling
description: Use when benchmarking the computational performance or scalability of
  LC-MS metabolomics data processing tools, and you need study-size-varied inputs
  from a fixed centroided mzML corpus rather than a single fixed-size run.
when_to_use_negative:
- The goal is feature-detection or quantification accuracy comparison (e.g. feature
  counts vs XCMS/MZmine), not runtime/memory scaling
- The input data are low-resolution or non-Orbitrap profiles without optimization,
  since asari's benchmark performance was established on high-resolution Orbitrap
  centroided mzML data
- Only one dataset size is needed and no scaling characterization is intended — a
  single timed run suffices
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: asari
  role: Tool under benchmark; run per subset via CLI (pip3 install asari-metabolomics)
    to measure CPU time and memory scaling
  repo: https://github.com/shuzhao-li/asari
- name: XCMS
  role: Reference tool benchmarked on the identical mzML subsets (peak calling and
    grouping in R/Bioconductor) to quantify the performance gap
- name: Python
  role: Resource-measurement wrapper (resource module or GNU time) capturing user
    CPU time, wall-clock time, and peak memory per run
- name: ThermoRawFileParser
  role: Converts Thermo .RAW files to .mzML so subsets consist of the centroided mzML
    inputs the tools require
  repo: https://github.com/compomics/ThermoRawFileParser
- name: msConvert
  role: Alternative converter for most vendor data formats and .mzXML into centroided
    mzML subsets
  repo: https://proteowizard.sourceforge.io/tools.shtml
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39889-1
    title: Trackable and scalable LC-MS metabolomics data processing using asari
  source_content_hash: sha256:b0ff149c7d0734a1d5b81614a2c3602996815834e41074f8e7a56b101c246615
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shuzhao-li/asari
  tools:
  - Python
  - ThermoRawFileParser
  - XCMS
  - asari
  - msConvert
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# mzml-dataset-subset-sampling

## Summary

Systematically varies LC-MS metabolomics study size by selecting nested subsets of centroided mzML files (e.g. 5, 10, 20, 50, 100 samples) from a public dataset, so that computational tools such as asari and XCMS can be benchmarked for CPU time, wall-clock time, and peak memory scaling.

## When to use

Use when benchmarking the computational performance or scalability of LC-MS metabolomics data processing tools, and you need study-size-varied inputs from a fixed centroided mzML corpus rather than a single fixed-size run.

## When NOT to use

- The goal is feature-detection or quantification accuracy comparison (e.g. feature counts vs XCMS/MZmine), not runtime/memory scaling
- The input data are low-resolution or non-Orbitrap profiles without optimization, since asari's benchmark performance was established on high-resolution Orbitrap centroided mzML data
- Only one dataset size is needed and no scaling characterization is intended — a single timed run suffices

## Inputs

- Public LC-MS metabolomics dataset in centroided mzML format (e.g. SLAW dataset subsets; example data at https://github.com/shuzhao-li-lab/data)
- List of subset sizes defining the sampling grid (e.g. 5, 10, 20, 50, 100 files)
- Optional vendor-format raw files (.RAW) plus a converter (ThermoRawFileParser or msConvert) to produce centroided mzML

## Outputs

- Nested mzML file subsets at each sample size, with identical files reused across all tools under test
- Per-tool, per-sample-size table of user CPU time, wall-clock time, and peak memory
- Scaling ratios and fitted/plot scaling curves used to characterize growth rate (linear vs super-linear)

## How to apply

Select subsets of increasing sample size (e.g. 5, 10, 20, 50, 100 files) from a public centroided mzML metabolomics dataset to vary study size systematically, keeping the instrument platform and acquisition parameters constant across subsets. Vendor data must first be converted to centroided mzML (e.g. ThermoRawFileParser for Thermo .RAW files or msConvert/ProteoWizard for other vendor formats), since the processing tools under test consume centroid mzML. Run the tool under test on each subset in a fresh process, wrapping each run with a resource-measurement harness such as the Python resource module or GNU time to capture user CPU time, wall-clock time, and peak memory. Tabulate measurements per tool per sample size, compute scaling ratios (time vs sample count), and fit/plot scaling curves to quantify the growth rate; in the asari paper this design demonstrated a 1-2 orders-of-magnitude CPU-time advantage over XCMS and largely linear scaling of CPU time and memory with sample number on SLAW dataset subsets.

## Related tools

- **asari** (Tool under benchmark; run per subset via CLI (pip3 install asari-metabolomics) to measure CPU time and memory scaling) — https://github.com/shuzhao-li/asari
- **XCMS** (Reference tool benchmarked on the identical mzML subsets (peak calling and grouping in R/Bioconductor) to quantify the performance gap)
- **Python** (Resource-measurement wrapper (resource module or GNU time) capturing user CPU time, wall-clock time, and peak memory per run)
- **ThermoRawFileParser** (Converts Thermo .RAW files to .mzML so subsets consist of the centroided mzML inputs the tools require) — https://github.com/compomics/ThermoRawFileParser
- **msConvert** (Alternative converter for most vendor data formats and .mzXML into centroided mzML subsets) — https://proteowizard.sourceforge.io/tools.shtml

## Examples

```
asari process --mode pos --input mydir/projectx_dir
```

## Evaluation signals

- Subset sizes increase monotonically (e.g. 5, 10, 20, 50, 100 files) and each subset is nested from the same source dataset so file difficulty is controlled
- All three metrics (user CPU time, wall-clock time, peak memory) are recorded for every tool × sample-size combination, with each tool run in a fresh process
- Identical mzML files (same conversion path, centroided) are fed to both tools so the comparison is like-for-like
- Fitted scaling curve for asari is approximately linear in sample number, matching the paper's SLAW-subset result of largely linear CPU-time and memory growth
- Performance gap vs XCMS falls in the reported range of roughly 1-2 orders of magnitude in CPU time

## Limitations

- Asari has been mostly tested on Orbitrap platforms; settings may need optimization for other instruments, which limits benchmark generalizability
- Asari does not work with low-resolution data and its efficiency is not optimal on data from longer chromatography
- MS/MS spectra are ignored by asari's default LC-MS workflow, so subsets must be profiled on MS1 feature processing only

## Evidence

- [readme] Input data are centroid mzML files from LC, GC or DI-MS metabolomics.: [evidence span withheld — anchor status: external_doc]
- [abstract] Asari offers substantial improvement in computational performance and is highly scalable: "It offers substantial improvement in computational performance over current tools, and it is highly scalable"
- [methods] Thermo RAW to mzML conversion requirement: "We use ThermoRawFileParser (https://github.com/compomics/ThermoRawFileParser) to convert Thermo .RAW files to .mzML"
