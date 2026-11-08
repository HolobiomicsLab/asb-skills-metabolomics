---
name: metabolomics-file-schedule-monitoring
description: Use when a QC4Metabolomics deployment (Docker container with continuous
  processing and module_File_schedule enabled) must be validated for near-real-time
  detection of newly acquired mzML/mzXML/mzData files, or when reporting maximum detection
  latency.
when_to_use_negative:
- Retrospective quality-control analysis of previously acquired files, which does
  not depend on near-real-time detection latency
- Evaluating module_TrackCmp or module_Contaminants peak-detection quality rather
  than the file-detection/scheduling latency itself
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: QC4Metabolomics
  role: Provides the file-scheduling pipeline (module_Files and module_File_schedule)
    whose near-real-time detection latency is being measured
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment platform for the QC4Metabolomics container and its docker-compose
    setup used to run the monitoring loop
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language of the QC4Metabolomics Shiny web application and R
    package containing the file-scheduling modules
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:dd0064ce086c494bf890035df24c2989e3d974b1fcf5ba647b480fc551661515
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metabolomics-file-schedule-monitoring

## Summary

Monitors QC4Metabolomics's near-real-time file detection loop by timing how quickly module_File_schedule picks up newly acquired mzML files added by module_Files, so agents can benchmark or debug latency in untargeted metabolomics QC.

## When to use

Use when a QC4Metabolomics deployment (Docker container with continuous processing and module_File_schedule enabled) must be validated for near-real-time detection of newly acquired mzML/mzXML/mzData files, or when reporting maximum detection latency.

## When NOT to use

- Retrospective quality-control analysis of previously acquired files, which does not depend on near-real-time detection latency
- Evaluating module_TrackCmp or module_Contaminants peak-detection quality rather than the file-detection/scheduling latency itself

## Inputs

- Newly acquired mzML files (also mzXML, mzData) placed in the QC4Metabolomics watched directory
- QC4Metabolomics Docker container with the continuous-processing process running and module_File_schedule enabled
- Demo data from the stanstrup/QC4Metabolomics repository
- Recorded timestamps of file arrival and of scheduling by module_File_schedule

## Outputs

- latency_report.csv with columns file, arrival_time, scheduled_time, latency_seconds
- Measured per-file detection latencies and the maximum value (expected of the order of up to 1 minute)

## How to apply

Start the QC4Metabolomics Docker container with the continuous-processing process running and module_File_schedule enabled (per the repository's docker-compose setup). Place a single mzML file (e.g. the repository's demo data) into the watched directory so module_Files detects it and adds the filename to the database, then record timestamps of file arrival and of when module_File_schedule reads the new filename and schedules downstream modules such as module_File_info. Repeat file placement several times and compute per-file latency as the difference between arrival and scheduling timestamps, retaining the maximum observed value. Summarize results in latency_report.csv with columns file, arrival_time, scheduled_time, latency_seconds; the paper states detection can take up to 1 minute, so values in that order confirm correct near-real-time behavior.

## Related tools

- **QC4Metabolomics** (Provides the file-scheduling pipeline (module_Files and module_File_schedule) whose near-real-time detection latency is being measured) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment platform for the QC4Metabolomics container and its docker-compose setup used to run the monitoring loop) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language of the QC4Metabolomics Shiny web application and R package containing the file-scheduling modules) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
docker compose up -d && cp demo.mzML /data/watched/ && date +%s > arrival.txt && Rscript -e 'QC4Metabolomics::schedule_check()' && date +%s > scheduled.txt
```

## Evaluation signals

- latency_report.csv exists with the exact columns file, arrival_time, scheduled_time, latency_seconds and one row per repeated file placement
- Maximum measured latency_seconds is of the order of up to 1 minute, consistent with near-real-time monitoring
- The scheduled file appears in downstream module processing (e.g. module_File_info) after module_File_schedule runs
- module_Files correctly registers the new mzML filename in the database before scheduling occurs
- Multiple repeated placements yield latencies within the expected sub-minute/≤1-minute range rather than sporadic long delays

## Limitations

- Detection of new files can take up to 1 minute, so near-real-time is not instantaneous
- Latency was characterized for the real-time single-file acquisition scenario; batch or multi-file arrivals may behave differently
- Processing overhead is slightly increased in the real-time single-file scenario, which can affect the observed timing

## Evidence

- [other] In the real-time single-file scenario, processing overhead is slightly increased and detection of new files can take up to 1 minute.: [evidence span withheld — anchor status: not_found]
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table: "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table"
- [methods] module_File_schedule checks if new filenames were added to the database: "module_File_schedule checks if new filenames were added to the database"
- [intro] monitors QC parameters in metabolomics experiments in (near) real-time: "monitors QC parameters in metabolomics experiments in (near) real-time"
