---
name: dockerized-instrument-data-pipeline-testing
description: Use when validating that an instrument data pipeline—specifically QC4Metabolomics's
  continuous Dockerized processing—detects newly acquired mzML files within the expected
  near-real-time window.
when_to_use_negative:
- Running retrospective QC on an already acquired batch of files—use the retrospective
  monitoring mode of QC4Metabolomics instead of testing real-time file detection latency
- Benchmarking CPU-intensive peak extraction (e.g. module_TrackCmp) rather than file-detection/scheduling
  latency
- Single-shot, non-Docker deployments where the continuous processing process and
  module_File_schedule are not running—file scheduling will not occur
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: 'The Dockerized QC pipeline being tested: modules module_Files and module_File_schedule
    detect and schedule new files for downstream processing'
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment runtime; docker-compose setup file spins up the QC4Metabolomics
    container, and ProteoWizard runs inside Docker to be OS-agnostic
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: QC4Metabolomics is an R package / Shiny web application; module logic and
    processing run in R
- name: ProteoWizard
  role: Converts vendor-specific raw data to mzML before module_Files detects it
- name: Shiny
  role: Web dashboard through which pipeline activity (e.g. module_Log) can be inspected
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:14a93bd819e1d4b565714a08b5a8eee23f38be93b446eb7c6bcd6f2281453a42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - ProteoWizard
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# dockerized-instrument-data-pipeline-testing

## Summary

Deploy QC4Metabolomics via its Docker images and docker-compose setup, place a demo mzML file into the watched directory, and measure the end-to-end latency of file detection by module_Files, scheduling by module_File_schedule, and downstream module processing.

## When to use

Use when validating that an instrument data pipeline—specifically QC4Metabolomics's continuous Dockerized processing—detects newly acquired mzML files within the expected near-real-time window.

## When NOT to use

- Running retrospective QC on an already acquired batch of files—use the retrospective monitoring mode of QC4Metabolomics instead of testing real-time file detection latency
- Benchmarking CPU-intensive peak extraction (e.g. module_TrackCmp) rather than file-detection/scheduling latency
- Single-shot, non-Docker deployments where the continuous processing process and module_File_schedule are not running—file scheduling will not occur

## Inputs

- mzML (or mzXML, mzData) files produced by converting vendor-specific instrument raw data via ProteoWizard
- demo data shipped with the QC4Metabolomics repository
- docker-compose setup file and Docker images from https://github.com/stanstrup/QC4Metabolomics
- watched directory or text file of file paths that module_Files traverses/reads

## Outputs

- Per-file latency measurements recorded as arrival_time, scheduled_time, and latency_seconds in latency_report.csv
- Maximum detection latency confirming near-real-time behavior (order of up to 1 minute)
- Evidence that module_File_schedule correctly scheduled the file for downstream modules such as module_File_info
- Log of QC4Metabolomics module activities (module_Log output)

## How to apply

Start the QC4Metabolomics Docker container with the continuous processing process running and module_File_schedule enabled, using the docker-compose setup file provided with the repository. Prepare the file-ingestion path exactly as in the article's architecture: copy vendor raw files to network storage via the Windows batch script intended for the instrument PC, convert vendor raw data to mzML with the automatic ProteoWizard converter (running in Docker and thus OS-agnostic), then have module_Files traverse a directory (or read from a text file of file paths) and add new mzML/mzXML/mzData filenames to a table in the database. Place a single mzML file (e.g. the repository demo data) into the watched directory, record timestamps for file arrival and for when module_File_schedule picks up the filename and schedules it for downstream processing such as module_File_info. Repeat several times and compute per-file latency as the difference between arrival and scheduling timestamps. Track the maximum observed latency and compare it to the paper's expectation that detection of new files can take up to 1 minute; in the real-time single-file scenario processing overhead is slightly increased, so any latency greatly above ~1 min or any missed file indicates a misconfigured watch path, database, or module ordering.

## Related tools

- **QC4Metabolomics** (The Dockerized QC pipeline being tested: modules module_Files and module_File_schedule detect and schedule new files for downstream processing) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment runtime; docker-compose setup file spins up the QC4Metabolomics container, and ProteoWizard runs inside Docker to be OS-agnostic) — https://github.com/stanstrup/QC4Metabolomics
- **R** (QC4Metabolomics is an R package / Shiny web application; module logic and processing run in R)
- **ProteoWizard** (Converts vendor-specific raw data to mzML before module_Files detects it)
- **Shiny** (Web dashboard through which pipeline activity (e.g. module_Log) can be inspected) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
docker-compose up -d && Rscript -e 'QC4Metabolomics::run_pipeline(mode="real-time", watch_dir="./mzml")'
```

## Evaluation signals

- Per-file latency = scheduled_time − arrival_time; the maximum across repeated placements should be of the order of up to 1 minute, matching the paper's real-time single-file scenario
- Every placed mzML file's filename appears in the database table that module_Files populates and is subsequently picked up by module_File_schedule
- Downstream module processing (e.g. module_File_info extracting instrument name, project name, ionization mode, sample name) completes without error for each scheduled file
- File conversion path works end-to-end: vendor raw data → ProteoWizard (in Docker) → mzML in the watched directory
- module_Log records the activities of the other modules with consistent timestamps for file detection and scheduling

## Limitations

- In the real-time single-file scenario, processing overhead is slightly increased and detection of new files can take up to 1 minute—latencies shorter than batch-style scanning or much longer than 1 min indicate a misconfiguration rather than expected behavior
- Only file formats traversed by module_Files (mzML, mzXML, mzData, etc.) are detected; vendor raw files must be converted first
- Pipeline operation depends on the instrument PC being able to copy files to a network drive accessible to QC4Metabolomics

## Evidence

- [other] In the real-time single-file scenario, processing overhead is slightly increased and detection of new files can take up to 1 minute.: [evidence span withheld — anchor status: not_found]
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table in the: "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table in the"
- [methods] An automatic converter that uses ProteoWizard, (in Docker and thus OS-agnostic) to convert vendor-specific raw data to mzML: "An automatic converter that uses ProteoWizard,
 (in Docker and thus OS-agnostic) to convert vendor-specific raw
data to mzML"
- [methods] A Windows batch script that is intended for use on the instrument PC to copy files to e.g. a network drive that QC4Metabolomics can access: "A Windows
batch script that is intended
for use on the instrument PC to copy files to e.g. a network drive
that QC4Metabolomics can access"
- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [readme] Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
