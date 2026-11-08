---
name: parallel-extraction-performance-measurement
description: Use when you must quantify whether Shinyscreen's mzML extraction and
  prescreening steps are accelerated by parallel backends — i.e. you have multiple
  vendor-neutral mzML files, a compound list, and need a reproducible serial vs parallel
  timing comparison with a speedup ratio.
when_to_use_negative:
- Benchmarking a single small mzML file, where future multisession worker start-up
  overhead can exceed any extraction gain
- Assessing Shinyscreen's interactive GUI, visualisation or MetFrag integration quality
  rather than extraction throughput
- The input is already a processed peak/feature table — no mzML extraction step remains
  to parallelise or time
tools:
- name: Shinyscreen
  role: Prescreening pipeline whose extraction and quality assessment stage is the
    workload being timed serially and in parallel
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: R
  role: Language used to script the timing harness (system.time) and assemble the
    CSV speedup table
- name: future
  role: Registers the parallel backend (e.g. multisession with N workers) under which
    the extraction pipeline is re-run
- name: MSnbase
  role: Underlying R infrastructure Shinyscreen relies on for parsing the mzML raw
    HRMS data during extraction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:f7502d13ea0c8eca44f122b95df9892e0e9b0ea9b1d82564a39d634ded479773
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - future
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# parallel-extraction-performance-measurement

## Summary

Benchmarks the wall-clock time of Shinyscreen's extraction and prescreening pipeline over mzML files run serially versus with a future-registered parallel backend, and reports the speedup ratio. This substantiates the paper's claim that Shinyscreen handles the thousands of HRMS peaks from high-throughput QC workloads efficiently.

## When to use

Use when you must quantify whether Shinyscreen's mzML extraction and prescreening steps are accelerated by parallel backends — i.e. you have multiple vendor-neutral mzML files, a compound list, and need a reproducible serial vs parallel timing comparison with a speedup ratio.

## When NOT to use

- Benchmarking a single small mzML file, where future multisession worker start-up overhead can exceed any extraction gain
- Assessing Shinyscreen's interactive GUI, visualisation or MetFrag integration quality rather than extraction throughput
- The input is already a processed peak/feature table — no mzML extraction step remains to parallelise or time

## Inputs

- Set of high-resolution MS mzML files (e.g. yeast amino-acid wild-type/knock-out runs)
- Compound list defining target precursor ion masses and adduct settings
- Extraction configuration: m/z tolerance, retention time error tolerance, MS1/MS2 intensity thresholds, retention time shifts
- Number of available CPU cores for future worker registration

## Outputs

- CSV timing table with columns serial_time_s, parallel_time_s, number_of_workers, speedup_ratio
- Identical extraction/prescreening result tables (EICs, MS2 scans, quality assessment calls) from serial and parallel runs
- system.time elapsed wall-clock measurements for each run

## How to apply

Load the same set of mzML files (e.g. the Saccharomyces cerevisiae amino-acid wild-type and knock-out teaching dataset) with the compound list, and configure identical extraction parameters in both runs: experimental m/z and retention time error tolerances, MS1 and MS2 intensity thresholds, and retention time shifts. Run the Shinyscreen extraction and prescreening pipeline serially — retrieving precursor ion masses from the compound list, extracted ion chromatograms (EICs) within the specified m/z tolerance, and corresponding MS2 spectral scans — recording elapsed wall-clock time with system.time. Then register a parallel backend with the future package (future::plan(multisession, workers = number of cores)) and re-run the identical pipeline, mirroring the paper's own parallelism strategy of running several instances in the background to speed up analysis. Assemble a timing table with serial_time_s, parallel_time_s, number_of_workers and speedup_ratio = serial_time_s / parallel_time_s. Validate that both runs consumed identical inputs and parameters and that the extracted EICs, MS2 scans and quality assessment results are unchanged between serial and parallel execution; the speedup ratio should exceed 1 for a parallelisation win.

## Related tools

- **Shinyscreen** (Prescreening pipeline whose extraction and quality assessment stage is the workload being timed serially and in parallel) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **R** (Language used to script the timing harness (system.time) and assemble the CSV speedup table)
- **future** (Registers the parallel backend (e.g. multisession with N workers) under which the extraction pipeline is re-run)
- **MSnbase** (Underlying R infrastructure Shinyscreen relies on for parsing the mzML raw HRMS data during extraction)

## Examples

```
future::plan(future::multisession, workers = parallel::detectCores()); t_par <- system.time(shinyscreen::run_prescreen(cfg))
```

## Evaluation signals

- Serial and parallel runs consumed byte-identical mzML files and identical extraction parameters (m/z and RT tolerances, MS1/MS2 intensity thresholds, RT shifts)
- Extracted EICs, MS2 spectral scans and quality-assessment calls are identical between the serial and parallel runs
- CSV contains the four required columns and speedup_ratio equals serial_time_s divided by parallel_time_s
- Recorded speedup_ratio is greater than 1, confirming parallelisation reduced processing time
- Timings are wall-clock (system.time) measurements on the same machine/worker configuration for both runs

## Limitations

- No archived code snapshot (Zenodo/OSF) of the timing workflow is reported, so exact reproduction depends on the GitLab repository state
- Speedup depends on workload size: with few or small mzML files, multisession worker overhead can erode or invert the ratio
- Timing is wall-clock and machine-dependent; results should be reported with the worker count and hardware context
- The parallel strategy applies to the background-capable extraction/analysis steps, not to interactive GUI inspection

## Evidence

- [abstract] High-throughput QC workload being accelerated: "a fully open source prescreening application to assist analysts in the high throughput quality control of the thousands of peaks detected in high resolution mass spectrometry experiments"
- [methods] Definition of the extraction step being timed: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved"
- [methods] Parallelism rationale in Shinyscreen: "Shinyscreen provides a MetFrag configuration interface (Fig. 5) that hides a lot of the complexity and runs several MetFrag instances in background to speed up the analysis"
- [methods] Core pipeline stages covered by the benchmark: "The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis"
- [methods] Extraction parameters that must be held identical across runs: "quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts"
