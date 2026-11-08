---
name: time-stamp-based-log-analysis
description: Use when you need to measure how quickly a continuous file-watching pipeline
  (e.g. QC4Metabolomics with module_File_schedule enabled) detects and schedules newly
  placed mzML files in near-real-time.
when_to_use_negative:
- You are evaluating retrospective QC dashboards or drift-correction quality (e.g.
  RT deviations, intensity drift) rather than file-detection latency
- Files are already present in the database from prior runs and you are not testing
  new-file detection behavior
- You need instrument-side acquisition alerts rather than pipeline-side file monitoring
  latency
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: Continuous processing pipeline whose module_Files and module_File_schedule
    modules are timed for new-file detection latency
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Runs the QC4Metabolomics container with continuous processing via the provided
    docker-compose setup
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Language in which QC4Metabolomics and its file-scheduling modules are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:26bb906ccb03d4e9c3f3d87bbf4c2593aee57035b72ae0b57b9e9e61725a6f8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# time-stamp-based-log-analysis

## Summary

Quantifies the file-detection latency of QC4Metabolomics' real-time monitoring by differencing timestamps of mzML file arrival versus module_File_schedule pickup, validating the paper's claim that new-file detection can take up to 1 minute.

## When to use

Use when you need to measure how quickly a continuous file-watching pipeline (e.g. QC4Metabolomics with module_File_schedule enabled) detects and schedules newly placed mzML files in near-real-time.

## When NOT to use

- You are evaluating retrospective QC dashboards or drift-correction quality (e.g. RT deviations, intensity drift) rather than file-detection latency
- Files are already present in the database from prior runs and you are not testing new-file detection behavior
- You need instrument-side acquisition alerts rather than pipeline-side file monitoring latency

## Inputs

- Single mzML files (mzML, mzXML, mzData etc.) placed into the QC4Metabolomics watched directory
- Timestamps of file arrival in the watched location
- Timestamps of filename pickup/scheduling by module_File_schedule from the database

## Outputs

- latency_report.csv with columns file, arrival_time, scheduled_time, latency_seconds
- Maximum and distribution of per-file detection latencies in seconds

## How to apply

Start the QC4Metabolomics Docker container with the continuous processing process running and module_File_schedule enabled, per the repository's docker-compose setup. Place single mzML files (e.g. the repository demo data) into the watched directory so module_Files traverses the directory, detects them, and adds filenames to the database. Record two timestamps per file: (a) when the mzML becomes available in the watched location and (b) when module_File_schedule picks up the new filename and schedules downstream processing (e.g. module_File_info). Repeat placement several times, compute per-file latency as scheduled_time minus arrival_time, and track the maximum observed value. Summarize into latency_report.csv with columns file, arrival_time, scheduled_time, latency_seconds.

## Related tools

- **QC4Metabolomics** (Continuous processing pipeline whose module_Files and module_File_schedule modules are timed for new-file detection latency) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Runs the QC4Metabolomics container with continuous processing via the provided docker-compose setup) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Language in which QC4Metabolomics and its file-scheduling modules are implemented)

## Examples

```
docker-compose up -d  # start QC4Metabolomics with module_File_schedule enabled, then place an mzML file in the watched dir and log arrival_time vs scheduled_time
```

## Evaluation signals

- Maximum measured detection latency is of the order of up to 1 minute, consistent with the paper's near-real-time monitoring claim
- Every placed mzML file eventually appears in the database and is scheduled for downstream modules (e.g. module_File_info)
- latency_report.csv contains non-negative latency_seconds for every repeated file-placement trial
- Latency overhead in the single-file scenario is only slightly increased relative to batch monitoring
- Docker container's continuous processing process and module_File_schedule are confirmed running for the duration of the test

## Limitations

- Detection of new files can take up to 1 minute, so the system is near-real-time rather than instantaneous
- Processing overhead is slightly increased in the real-time single-file scenario compared to retrospective processing
- Results depend on the continuous processing process and module_File_schedule being enabled in the Docker container; the README only points to external documentation (http://stanstrup.github.io/QC4Metabolomics/)

## Evidence

- [results] In the real-time single-file scenario, processing overhead is slightly increased and detection of new files can take up to 1 minute.: [evidence span withheld — anchor status: not_found]
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths: "module_Files looks for newly acquired
files (mzML, mzXML, mzData etc.) either by traversing a directory
or by reading from a text file containing file paths"
- [methods] module_File_schedule checks if new filenames were added to the database: "module_File_schedule checks if new
filenames were added to the database"
- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [intro] monitors QC parameters in metabolomics experiments in (near) real-time: "monitors QC parameters in metabolomics experiments in (near) real-time"
