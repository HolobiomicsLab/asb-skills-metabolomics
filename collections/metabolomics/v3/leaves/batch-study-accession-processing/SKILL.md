---
name: batch-study-accession-processing
description: Use when benchmarking whole-process conversion correctness across a cohort
  of study accessions (e.g. 41 accepted data sets or 52 repository studies), needing
  per-study success/failure status plus aggregate correct-output percentages.
when_to_use_negative:
- The data are the user's own private/local files — CloMet handles these via the LOCAL
  tab rather than study-id-based repository retrieval
- The goal is statistical analysis or data processing — CloMet does not implement
  data processing algorithms; it only harmonizes formats for downstream tools
- A single data set needs interactive conversion rather than a batch of study accessions
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: CloMet
  role: Runs the LoadDataset pipeline per study ID to download raw/NMR data and convert
    it into the analysis-ready output format
  repo: https://github.com/rmallol/clomet
- name: Docker
  role: Execution container in which each LoadDataset run is performed
- name: MetaboLights
  role: Source repository for one cohort of study accessions
- name: Metabolomics Workbench
  role: Source repository for one cohort of study accessions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:c36b177eb61d9ba0b1377b777454f6f176b7bc8149980fccb3a0b973c5b008bc
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Docker
  - MetaboLights
  - Metabolomics Workbench
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# batch-study-accession-processing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Batch-run the CloMet LoadDataset pipeline inside Docker for a list of MetaboLights/Metabolomics Workbench study IDs, judging each run as a correct ready-to-use output or a failure, then aggregate success rates (e.g. 38/41 = 92.7% for accepted data sets; 38/52 = 73.1% for repository studies, Figures 8B/8C).

## When to use

Use when benchmarking whole-process conversion correctness across a cohort of study accessions (e.g. 41 accepted data sets or 52 repository studies), needing per-study success/failure status plus aggregate correct-output percentages.

## When NOT to use

- The data are the user's own private/local files — CloMet handles these via the LOCAL tab rather than study-id-based repository retrieval
- The goal is statistical analysis or data processing — CloMet does not implement data processing algorithms; it only harmonizes formats for downstream tools
- A single data set needs interactive conversion rather than a batch of study accessions

## Inputs

- List of study IDs (accession identifiers) from MetaboLights and/or Metabolomics Workbench
- CloMet Docker container with the LoadDataset pipeline
- Cohort definition (e.g. 41 accepted data sets; 52 repository studies)

## Outputs

- Per-study conversion status (correct vs failed) for each run
- benchmark_results.csv with per-study statuses and aggregate correct-output rates (percentages)
- Analysis-ready output data sets in the format usable by MetaboAnalyst / Workflows4Metabolomics

## How to apply

Retrieve the list of study IDs defining your cohort (e.g. the 41 accepted data sets and the 52 studies from MetaboLights and Metabolomics Workbench). For each study ID, run the CloMet LoadDataset pipeline inside the provided Docker container to download the raw/NMR data and convert it into the analysis-ready output format. Inspect each of the runs to decide whether the conversion produced a correct, ready-to-use output data set usable in MetaboAnalyst or Workflows4Metabolomics. Aggregate correct counts per cohort, compute success percentages, and write per-study statuses plus aggregate rates to benchmark_results.csv. Validate the reproduction by checking counts against the reported figures (38/41 = 92.7% and 38/52 = 73.1%).

## Related tools

- **CloMet** (Runs the LoadDataset pipeline per study ID to download raw/NMR data and convert it into the analysis-ready output format) — https://github.com/rmallol/clomet
- **Docker** (Execution container in which each LoadDataset run is performed)
- **MetaboLights** (Source repository for one cohort of study accessions)
- **Metabolomics Workbench** (Source repository for one cohort of study accessions)

## Evaluation signals

- Counts of correct outputs match reported rates: 38/41 (92.7%) for accepted data sets and 38/52 (73.1%) for repository studies (Figures 8B and 8C)
- Every one of the 93 study-ID runs has an explicit correct/failed status recorded
- benchmark_results.csv contains both per-study statuses and aggregate percentages for the two cohorts
- Successful conversions produce output data sets directly loadable in MetaboAnalyst or Workflows4Metabolomics (e.g. NMR_Read, NMR_Preprocessing, NMR_Bucketing tools)

## Limitations

- Failures occur when class metadata is unavailable — re-analysis oriented to classification problems won't be feasible for those data sets
- CloMet does not itself implement data processing algorithms; conversion success is the only quality metric measured
- Maintenance and extension rely on the open-source developer community, so pipeline behavior may drift
- Not all runs succeed — 5 of 41 accepted data sets (and 14 of 52 repository studies) failed to convert in the reference reproduction

## Evidence

- [abstract] CloMet converts raw and NMR-based metabolomics data from MetaboLights and Metabolomics Workbench to a file format that can be used directly: "converts raw and NMR-based metabolomics data from MetaboLights and Metabolomics Workbench to a file format that can be used directly"
- [discussion] study id retrieval yields ready-to-use data set: "by just inserting a study id our users can get a ready-to-use data set with the minimal interaction possible"
- [other] Docker LoadDataset run per study ID: [evidence span withheld — anchor status: not_found]
- [readme] README purpose statement: "CloMet eases the connection between public data repositories and data analysis platforms by harmonizing the file systems of available data sets"
