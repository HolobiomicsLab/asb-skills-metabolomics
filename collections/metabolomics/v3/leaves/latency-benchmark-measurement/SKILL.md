---
name: latency-benchmark-measurement
description: Use when you need to quantify the near-real-time responsiveness of QC4Metabolomics'
  file-monitoring loop — i.e. benchmark detection latency for newly acquired mzML/mzXML/mzData
  files under the Docker-based continuous-processing setup.
when_to_use_negative:
- When you want to benchmark downstream processing cost (module_File_info, module_TrackCmp,
  module_Contaminants) rather than the file-detection delay itself
- When QC4Metabolomics is run in purely retrospective mode — the 1-minute detection
  latency applies to the real-time single-file scenario where files are monitored
  as they are acquired, not to batch post-analysis
- When the watched directory or the text file of file paths is not being updated at
  all; then the failure is a configuration error, not a latency characteristic
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: The software under test; module_Files detects new mzML/mzXML/mzData files
    and module_File_schedule checks for new filenames added to the database and schedules
    them for downstream modules.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment container used to launch the QC4Metabolomics continuous-processing
    process via the provided docker-compose setup file.
- name: R
  role: Language of the QC4Metabolomics R package and Shiny app that implement the
    monitoring pipeline being timed.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:85cb79f379a20d814103013205a7320d79b35b687b0f55291820112e98bb0c66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# latency-benchmark-measurement

## Summary

Measures how long QC4Metabolomics takes to notice a newly acquired mzML file by timestamping file arrival in the watched directory versus the moment module_File_schedule schedules it, confirming the paper's claim that detection can take up to 1 minute.

## When to use

Use when you need to quantify the near-real-time responsiveness of QC4Metabolomics' file-monitoring loop — i.e. benchmark detection latency for newly acquired mzML/mzXML/mzData files under the Docker-based continuous-processing setup.

## When NOT to use

- When you want to benchmark downstream processing cost (module_File_info, module_TrackCmp, module_Contaminants) rather than the file-detection delay itself
- When QC4Metabolomics is run in purely retrospective mode — the 1-minute detection latency applies to the real-time single-file scenario where files are monitored as they are acquired, not to batch post-analysis
- When the watched directory or the text file of file paths is not being updated at all; then the failure is a configuration error, not a latency characteristic

## Inputs

- A running QC4Metabolomics Docker container with the continuous processing process and module_File_schedule enabled (docker-compose setup)
- mzML (or mzXML/mzData) files to drop into the watched directory — e.g. the demo data shipped with the repo
- Instrument PC Windows batch script copying vendor raw files to the network drive that QC4Metabolomics accesses, and ProteoWizard-based automatic conversion to mzML if starting from vendor raw data

## Outputs

- latency_report.csv with columns file, arrival_time, scheduled_time, latency_seconds
- A summary of measured per-file detection latencies, including the maximum observed value

## How to apply

Start the QC4Metabolomics Docker container with the continuous processing process and module_File_schedule enabled, using the repository's docker-compose setup. Place an mzML file (e.g. the bundled demo data) into the watched directory and record two timestamps: (a) when the file becomes available to the filesystem and (b) when module_File_schedule picks the filename added by module_Files from the database and schedules downstream modules such as module_File_info. Repeat the placement several times so per-file latencies (arrival_time minus scheduled_time, in seconds) can be computed, then capture the maximum. Validation is the maximum latency landing in the order of up to 1 minute, matching the paper's description of near-real-time file monitoring; persist results to latency_report.csv with columns file, arrival_time, scheduled_time, latency_seconds.

## Related tools

- **QC4Metabolomics** (The software under test; module_Files detects new mzML/mzXML/mzData files and module_File_schedule checks for new filenames added to the database and schedules them for downstream modules.) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment container used to launch the QC4Metabolomics continuous-processing process via the provided docker-compose setup file.)
- **R** (Language of the QC4Metabolomics R package and Shiny app that implement the monitoring pipeline being timed.)

## Examples

```
docker-compose up -d && cp demo.mzML /data/watched/ && python measure_latency.py --watch /data/watched --out latency_report.csv
```

## Evaluation signals

- The maximum latency_seconds in latency_report.csv is of the order of up to 1 minute (~60 s) rather than hours or seconds.
- Every placed file has both arrival_time and scheduled_time filled; no file is missing a scheduling event, indicating module_Files and module_File_schedule both fired.
- Per-file latency_seconds is non-negative and roughly consistent across repeats (variation confined to the ~1-minute window).
- Detection latency is measured in the single-file real-time scenario — one file dropped at a time into the watched directory — matching the paper's experimental setup.

## Limitations

- The paper states detection of new files 'can take up to 1 minute' in the real-time single-file scenario, so sub-minute real-time guarantees are not supported by this pipeline.
- Processing overhead is slightly increased in the real-time single-file scenario, so latency measurements conflate detection delay with that overhead.
- The measurement depends on module_Files traversing the watched directory (or reading a text file of paths) and module_File_schedule running continuously; any stall in those modules invalidates the benchmark.

## Evidence

- [results] In the real-time single-file scenario, processing overhead is slightly increased and detection of new files can take up to 1 minute.: [evidence span withheld — anchor status: not_found]
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table in the: "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths"
- [intro] monitors QC parameters in metabolomics experiments in (near) real-time: "monitors QC parameters in metabolomics experiments in (near) real-time"
- [methods] module_File_schedule checks if new filenames were added to the database: "module_File_schedule checks if new filenames were added to the database"
