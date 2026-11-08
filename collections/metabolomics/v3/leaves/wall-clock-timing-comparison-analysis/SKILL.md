---
name: wall-clock-timing-comparison-analysis
description: Use when profiling the runtime of an R mass-spectrometry pipeline (e.g.
  Shinyscreen's mzML extraction and prescreening) to quantify how much parallelisation
  reduces processing time.
when_to_use_negative:
- The goal is compound identification or spectrum annotation rather than quantifying
  wall-clock speedup of the extraction step.
- The workflow is a single-file, interactive exploratory run where parallel overhead
  would dwarf any gain.
- Input is not vendor-independent mzML data amenable to Shinyscreen's list-based (compound
  or mass) screening.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0003
- http://edamontology.org/topic_3520
tools:
- name: Shinyscreen
  role: The prescreening application whose extraction and quality-assessment pipeline
    is timed; supports compound list-based and mass list-based screening of HRMS peaks.
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: future
  role: Supplies the parallel backend (e.g. multisession with a set number of workers)
    used for the parallel timing run.
- name: MSnbase
  role: R package Shinyscreen builds on for handling the mzML spectra during extraction.
- name: R
  role: Runtime environment hosting Shinyscreen, MSnbase and future; system.time and
    CSV export are done here.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:1d24e596304308eb3077f3de7641b9ad9c6c6642f1566a2fb68df28dc7477061
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - future
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0003
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# wall-clock-timing-comparison-analysis

## Summary

Benchmarks Shinyscreen's high-resolution MS prescreening/extraction pipeline by running the identical workload twice — once serially and once with a future-parallel backend — and reporting elapsed wall-clock times and speedup ratios.

## When to use

Use when profiling the runtime of an R mass-spectrometry pipeline (e.g. Shinyscreen's mzML extraction and prescreening) to quantify how much parallelisation reduces processing time.

## When NOT to use

- The goal is compound identification or spectrum annotation rather than quantifying wall-clock speedup of the extraction step.
- The workflow is a single-file, interactive exploratory run where parallel overhead would dwarf any gain.
- Input is not vendor-independent mzML data amenable to Shinyscreen's list-based (compound or mass) screening.

## Inputs

- yeast amino-acid high-resolution MS mzML files
- compound list with precursor ion masses/adduct settings
- extraction configuration (m/z and RT errors, MS1/MS2 intensity thresholds, RT shifts)
- number of worker processes for future backend

## Outputs

- CSV timing table with serial_time_s, parallel_time_s, number_of_workers and speedup_ratio
- Shinyscreen extraction/prescreening results (EICs, MS2 spectra, quality assessment) for each run

## How to apply

Load the yeast amino-acid mzML files and compound list, then run the Shinyscreen initialisation–extraction–prescreening pipeline serially, wrapping the call in system.time to capture elapsed seconds. Register a parallel backend with future (e.g. future::plan(multisession, workers = n_cores)) and re-run the identical extraction and prescreening workload on the same mzML files with the same m/z and retention-time errors, MS1/MS2 intensity thresholds and retention-time shifts. Record serial_time_s, parallel_time_s, number_of_workers and the speedup ratio (serial_time_s / parallel_time_s) in a CSV table. Validate that both runs used identical input files and parameters and that the speedup ratio exceeds 1, confirming parallelisation reduces processing time. The workflow relies on Shinyscreen retrieving precursor-ion masses, EICs within the specified m/z tolerance and corresponding MS2 spectral scans from the mzML files before performing the quality assessment stage.

## Related tools

- **Shinyscreen** (The prescreening application whose extraction and quality-assessment pipeline is timed; supports compound list-based and mass list-based screening of HRMS peaks.) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **future** (Supplies the parallel backend (e.g. multisession with a set number of workers) used for the parallel timing run.)
- **MSnbase** (R package Shinyscreen builds on for handling the mzML spectra during extraction.)
- **R** (Runtime environment hosting Shinyscreen, MSnbase and future; system.time and CSV export are done here.)

## Examples

```
Rscript -e "plan(multisession, workers=n); s1 <- system.time(extract_prescreen(mzmls)); s2 <- system.time(extract_prescreen(mzmls)); write.csv(data.frame(serial=s1[[3]], parallel=s2[[3]], workers=n, speedup=s1[[3]]/s2[[3]]), 'timing.csv')"
```

## Evaluation signals

- Both timed runs consume the identical set of mzML files and identical extraction parameters (m/z/RT errors, MS1/MS2 thresholds, RT shifts).
- The CSV table contains serial_time_s, parallel_time_s, number_of_workers and speedup_ratio columns for every workload.
- speedup_ratio = serial_time_s / parallel_time_s and its value is greater than 1.
- system.time is used on both runs so elapsed wall-clock time is captured consistently.
- The parallel run reports the registered future backend (e.g. multisession) and the worker count used.

## Limitations

- Parallel speedup applies to Shinyscreen's extraction and prescreening stage; downstream visualisation or MetFrag integration steps are not covered by this benchmark.
- Results depend on the chosen worker count and machine configuration; speedup may vary or degrade with too many workers.
- Timing is specific to the yeast amino-acid teaching dataset; other mzML inputs (different file sizes, scan counts) will yield different ratios.

## Evidence

- [methods] extraction timing method: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans are retrieved from the"
- [methods] quality assessment stage: "the data is subjected to a quality assessment stage. The quality of each MS1 parent data point is assessed based on the existence of MS1 and MS2 data, the noise level and the retention time shift"
- [methods] core pipeline steps being timed: "The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis"
- [methods] dataset used for timing: "targeted and non-targeted analysis of amino acids in Saccharomyces cerevisiae (yeast) wild-type and knock-out strains. This data set also serves as the test data provided in the package vignette"
- [abstract] pipeline purpose motivating the timing test: "a fully open source prescreening application to assist analysts in the high throughput quality control of the thousands of peaks detected in high resolution mass spectrometry experiments"
