---
name: hrms-data-extraction-benchmarking
description: Use when a Shinyscreen or MSnbase-based mzML extraction/prescreening
  workload runs too slowly serially and you need to quantify wall-clock speedup from
  registering a future parallel backend.
when_to_use_negative:
- When the goal is compound identification or spectra annotation rather than benchmarking
  — use Shinyscreen's MetFrag integration instead
- When the workload is a single mzML file or very few files, where parallel backend
  setup overhead can erase any speedup
- When the serial and parallel runs would use different input files or extraction
  parameters — the speedup ratio is only valid for an identical workload
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Shinyscreen
  role: Runs the extraction and prescreening pipeline (EIC and MS2 retrieval, quality
    assessment) whose serial vs parallel wall-clock times are benchmarked
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: future
  role: Registers the parallel backend (e.g. multisession with N workers) used for
    the re-run of the identical extraction workload
- name: MSnbase
  role: Underlying R infrastructure for flexible analysis of the raw MS data that
    Shinyscreen's extraction reads from the mzML files
- name: R
  role: Implementation language in which the pipeline, system.time timings, and CSV
    timing table are produced
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:ad966fd56ded8e9ae5bd52f00a4e802cf150a880a9c784ae607c6d582b797a73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - future
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hrms-data-extraction-benchmarking

## Summary

Benchmark serial versus future-parallel execution of Shinyscreen's HRMS extraction and prescreening pipeline (EIC and MS2 retrieval from mzML files) to quantify wall-clock speedup for high-throughput quality control of thousands of peaks.

## When to use

Use when a Shinyscreen or MSnbase-based mzML extraction/prescreening workload runs too slowly serially and you need to quantify wall-clock speedup from registering a future parallel backend.

## When NOT to use

- When the goal is compound identification or spectra annotation rather than benchmarking — use Shinyscreen's MetFrag integration instead
- When the workload is a single mzML file or very few files, where parallel backend setup overhead can erase any speedup
- When the serial and parallel runs would use different input files or extraction parameters — the speedup ratio is only valid for an identical workload

## Inputs

- mzML files of high resolution mass spectrometry runs (e.g. Saccharomyces cerevisiae amino-acid wild-type and knock-out strains)
- compound list with precursor/adduct information for targeted extraction
- extraction/prescreening configuration: m/z tolerance, retention time error, MS1 and MS2 intensity thresholds, retention time shifts

## Outputs

- CSV timing table with serial_time_s, parallel_time_s, number_of_workers, and speedup_ratio (serial_time_s / parallel_time_s)
- identical extraction and prescreening outputs from both runs (EICs, MS2 spectral scans, quality assessment results) for validation

## How to apply

Load the HRMS mzML files (e.g. the Saccharomyces cerevisiae amino-acid teaching dataset) plus the compound list, and configure extraction parameters: experimental m/z and retention time error tolerances, MS1 and MS2 intensity thresholds, and retention time shifts. Run Shinyscreen's extraction and prescreening pipeline serially — retrieval of precursor ion masses from the compound lists, extracted ion chromatograms (EICs within the specified m/z tolerance), and corresponding MS2 spectral scans — recording elapsed wall-clock time with system.time. Then register a parallel backend with future::plan(multisession, workers = number of cores) and re-run the identical extraction and prescreening pipeline on the same files, again timing with system.time. Assemble a CSV timing table with columns serial_time_s, parallel_time_s, number_of_workers, and speedup_ratio (serial_time_s / parallel_time_s) for the identical workload. The rationale is the same parallelisation strategy the paper applies elsewhere: Shinyscreen hides complexity and 'runs several ... instances in background to speed up the analysis', so background execution via the future package should reduce wall-clock time on multi-file workloads.

## Related tools

- **Shinyscreen** (Runs the extraction and prescreening pipeline (EIC and MS2 retrieval, quality assessment) whose serial vs parallel wall-clock times are benchmarked) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **future** (Registers the parallel backend (e.g. multisession with N workers) used for the re-run of the identical extraction workload)
- **MSnbase** (Underlying R infrastructure for flexible analysis of the raw MS data that Shinyscreen's extraction reads from the mzML files)
- **R** (Implementation language in which the pipeline, system.time timings, and CSV timing table are produced)

## Examples

```
future::plan(multisession, workers = parallel::detectCores()); system.time(run_extraction_and_prescreening(config))
```

## Evaluation signals

- Both runs used identical input mzML files, compound lists, and extraction parameters (m/z tolerance, RT error, MS1/MS2 intensity thresholds, RT shifts)
- Recorded speedup_ratio (serial_time_s / parallel_time_s) is greater than 1, confirming parallelisation reduces processing time
- The CSV timing table is complete with serial_time_s, parallel_time_s, number_of_workers, and speedup_ratio columns
- Wall-clock times were captured with system.time for both the serial and future-parallel executions
- Extraction outputs (EICs, MS2 spectral scans, quality assessment results) are equivalent between the serial and parallel runs

## Limitations

- Speedup depends on the number of mzML files and available cores; small workloads may show no benefit from multisession parallelisation
- License is Apache 2.0 but no archived code snapshot (Zenodo/OSF) is reported, so exact pipeline versions used in timing may not be reproducible
- No data accession or dataset identifier is reported for the teaching dataset used, complicating exact replication of the benchmark

## Evidence

- [abstract] Shinyscreen is a fully open source prescreening application to assist analysts in the high throughput quality control of the thousands of peaks detected in high resolution mass spectrometry experiments.: "a fully open source prescreening application to assist analysts in the high throughput quality control of the thousands of peaks"
- [methods] Extraction retrieves precursor ion masses, EICs within the m/z tolerance, and MS2 scans from the raw data.: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as"
- [methods] QC parameters include MS1/MS2 intensity thresholds and retention time shifts that must be held fixed across benchmark runs.: "quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts"
- [methods] Shinyscreen already uses background parallel execution to speed up analysis, motivating the future-backend benchmark.: "Shinyscreen provides a MetFrag configuration interface (Fig. 5) that hides a lot of the complexity and runs several MetFrag instances"
