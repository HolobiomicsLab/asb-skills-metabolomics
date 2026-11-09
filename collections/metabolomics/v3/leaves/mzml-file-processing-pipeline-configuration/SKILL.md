---
name: mzml-file-processing-pipeline-configuration
description: Use when raw HRMS instrument files in mzML format must be extracted against
  a compound or mass list with configured m/z/RT tolerances and MS1/MS2 intensity
  thresholds, especially when serial vs parallel wall-clock timing must be compared.
when_to_use_negative:
- The data is already a processed feature table (e.g. from xcms or RforMassSpectrometry)
  rather than raw mzML files requiring extraction
- The goal is interactive low-level access to individual spectra or peaks without
  a compound/mass list — low-level packages such as mzR are the article-cited fit
  for that case
- Compound identification downstream of prescreening is the goal — use Shinyscreen's
  MetFrag integration rather than bare extraction configuration
edam_operation: http://edamontology.org/operation_0335
edam_topics: []
tools:
- name: Shinyscreen
  role: R package / Shiny web application whose initialisation and extraction-prescreening
    stages are configured and run over the mzML files
  repo: https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- name: MSnbase
  role: Underlying R package for analysis of MS data cited as the flexible backend
    Shinyscreen sits above for mzML handling
- name: future
  role: Registers a parallel backend (e.g. plan(multisession, workers = n)) so the
    identical extraction pipeline can be re-run in parallel for timing comparison
- name: R
  role: Runtime environment in which the pipeline configuration, extraction, and system.time
    timing measurements are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-025-01044-x
    title: 'Shinyscreen: mass spectrometry data inspection and quality checking utility'
  source_content_hash: sha256:888a469c888e119979faac5dcc5e8b87a13e7826acaf91279163c780217d3272
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MSnbase
  - R
  - Shinyscreen
  - future
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mzml-file-processing-pipeline-configuration

## Summary

Configure and execute Shinyscreen's initialisation → extraction/prescreening pipeline over raw high-resolution MS data in mzML format, setting m/z and retention-time error tolerances, MS1/MS2 intensity thresholds and retention-time shifts, and optionally parallelising the identical extraction run with the R future package.

## When to use

Use when raw HRMS instrument files in mzML format must be extracted against a compound or mass list with configured m/z/RT tolerances and MS1/MS2 intensity thresholds, especially when serial vs parallel wall-clock timing must be compared.

## When NOT to use

- The data is already a processed feature table (e.g. from xcms or RforMassSpectrometry) rather than raw mzML files requiring extraction
- The goal is interactive low-level access to individual spectra or peaks without a compound/mass list — low-level packages such as mzR are the article-cited fit for that case
- Compound identification downstream of prescreening is the goal — use Shinyscreen's MetFrag integration rather than bare extraction configuration

## Inputs

- Raw HRMS data files in mzML format (e.g. Saccharomyces cerevisiae amino-acid wild-type and knock-out strain runs)
- Compound list containing target structures and adduct settings, from which precursor ion masses are calculated
- Configuration specifying m/z and retention-time error tolerances, MS1/MS2 intensity thresholds, and retention-time shifts
- Choice of parallel backend and worker count for the future package (e.g. multisession with n workers)

## Outputs

- Extracted ion chromatograms (EICs) and MS2 spectral scans per compound/mass entry
- Prescreening and quality-assessment tables for MS1 parent data points (existence of MS1/MS2 data, noise level, retention-time shift)
- CSV timing table with columns serial_time_s, parallel_time_s, number_of_workers and speedup_ratio

## How to apply

Start from Shinyscreen's initialisation step: load the mzML files and the compound list, and calculate precursor ion masses automatically from the given adduct settings. Configure extraction parameters — experimental m/z and retention-time errors, MS1 and MS2 intensity thresholds, and retention-time shifts — then run the extraction and prescreening stage, in which the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) and corresponding MS2 spectral scans are retrieved. Run the identical pipeline first serially (no parallel backend), recording elapsed wall-clock time with system.time, then register a parallel backend with future (e.g. plan(multisession, workers = n)) and re-run the same extraction to obtain a comparable timing. Assemble a CSV timing table with serial_time_s, parallel_time_s, number_of_workers and speedup_ratio (serial_time_s / parallel_time_s); a ratio greater than 1 confirms parallelisation reduces processing time. Validate that both runs used byte-identical input files and parameters so the speedup is attributable to the backend, not to configuration drift.

## Related tools

- **Shinyscreen** (R package / Shiny web application whose initialisation and extraction-prescreening stages are configured and run over the mzML files) — https://gitlab.com/uniluxembourg/lcsb/eci/shinyscreen
- **MSnbase** (Underlying R package for analysis of MS data cited as the flexible backend Shinyscreen sits above for mzML handling)
- **future** (Registers a parallel backend (e.g. plan(multisession, workers = n)) so the identical extraction pipeline can be re-run in parallel for timing comparison)
- **R** (Runtime environment in which the pipeline configuration, extraction, and system.time timing measurements are executed)

## Examples

```
future::plan(multisession, workers = 8); system.time(shinyscreen_prescreen(config))  # run once without plan() for serial timing
```

## Evaluation signals

- Serial and parallel runs consumed identical mzML input files and identical extraction parameters (m/z and RT errors, MS1/MS2 intensity thresholds, retention-time shifts) — verify via configuration diff
- The recorded speedup_ratio (serial_time_s / parallel_time_s) is greater than 1, confirming parallelisation reduced processing time
- Retrieved EICs fall within the specified m/z tolerance around the precursor ion masses calculated from the adduct settings
- The timing CSV contains all four required columns (serial_time_s, parallel_time_s, number_of_workers, speedup_ratio) for the identical workload
- Prescreening output covers every compound/mass entry in the input list across all supplied mzML files

## Limitations

- Parallel speedup depends on a registered future backend being available; with no backend, the pipeline runs serially and no speedup is realised
- License (Apache 2.0) is stated but no archived code snapshot (Zenodo/OSF) is reported in the article, so exact code provenance must be taken from the GitLab repository
- No data accession or dataset identifier is reported for the teaching dataset, so the exact mzML inputs must be obtained from the package vignette test data

## Evidence

- [abstract] abstract: "Shinyscreen is an R package and Shiny-based web application designed for the exploration, visualization, and quality assessment of raw data from high resolution mass spectrometry instruments."
- [methods] methods_pipeline_steps: "The core steps of Shinyscreen include initialisation, extraction and prescreening, visualisation and integration with MetFrag for further analysis"
- [methods] methods_extraction: "the precursor ion masses from the compound lists, their extracted ion chromatograms (EICs, within the specified m/z tolerance) as well as their corresponding MS2 spectral scans"
- [methods] methods_adducts: "Shinyscreen will automatically calculate required precursor ion masses for extraction, using the given adduct setting"
- [methods] methods_qc_params: "quality control parameters such as MS1 and MS2 intensity thresholds and retention time shifts"
- [methods] methods_tolerances: "parametrise (a) experimental m/z and retention time errors"
- [methods] methods_dataset: "targeted and non-targeted analysis of amino acids in Saccharomyces cerevisiae (yeast) wild-type and knock-out strains"
