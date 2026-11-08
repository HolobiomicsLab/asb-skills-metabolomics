---
name: per-file-processing-timing-benchmarking
description: Use when you need to quantify per-file processing cost of QC4Metabolomics
  — e.g. estimating how many mzML files the pipeline can handle per hour or planning
  acquisition scheduling around module latency.
when_to_use_negative:
- The goal is end-to-end QC assessment rather than timing — QC4Metabolomics is meant
  to monitor QC parameters, not to benchmark throughput.
- The input data are not mzML/mzXML/mzData metabolomics files, so module_File_info
  and module_TrackCmp have nothing to extract.
- Absolute timing accuracy is needed on a shared/loaded host; host machine load and
  hardware make per-file times only order-of-magnitude comparable.
tools:
- name: QC4Metabolomics
  role: pipeline under test — runs module_File_info, module_TrackCmp and module_Contaminants
    on each demo mzML file
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: deployment mechanism for the QC4Metabolomics pipeline (docker-compose setup
    with demo data)
- name: R
  role: runtime of the QC4Metabolomics processing modules being timed
- name: Shiny
  role: web front-end of the QC4Metabolomics application whose modules are benchmarked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:9395391637eaac705a59ba213065bb6852c3e28e425df65a3133bbe6c383f670
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-file-processing-timing-benchmarking

## Summary

Benchmarks the wall-clock time of each QC4Metabolomics processing module per mzML file (quality-parameter extraction and contaminant screening) by running the Dockerized pipeline on bundled demo data and recording timings to CSV.

## When to use

Use when you need to quantify per-file processing cost of QC4Metabolomics — e.g. estimating how many mzML files the pipeline can handle per hour or planning acquisition scheduling around module latency.

## When NOT to use

- The goal is end-to-end QC assessment rather than timing — QC4Metabolomics is meant to monitor QC parameters, not to benchmark throughput.
- The input data are not mzML/mzXML/mzData metabolomics files, so module_File_info and module_TrackCmp have nothing to extract.
- Absolute timing accuracy is needed on a shared/loaded host; host machine load and hardware make per-file times only order-of-magnitude comparable.

## Inputs

- Demo mzML files bundled with QC4Metabolomics (also any mzML, mzXML, or mzData files detected by module_Files)
- User-defined compound list of m/z and retention time (RT) pairs for module_TrackCmp
- Built-in contaminant ion list (~800 ions) used by module_Contaminants

## Outputs

- Per-file wall-clock timings for quality-parameter extraction (s/file)
- Per-file wall-clock timings for contaminant screening (s/file)
- Aggregated CSV (processing_timings.csv) with the measured timings

## How to apply

Deploy QC4Metabolomics via its docker-compose setup with the bundled demo mzML files enabled, and start the continuously running processing process so module_Files detects the demo files and module_File_schedule queues them. Let module_File_info extract metadata (instrument name, project name, ionization mode, sample name) and module_TrackCmp extract m/z, retention time, intensity, and peak-shape diagnostics for the user-defined compound list; then let module_Contaminants screen each file against the built-in list of nearly 800 known contaminant ions. Time each step per file with a wall-clock timer and write the per-file values to processing_timings.csv. Validate that measured timings are of the same order as the reported figures — quality-parameter extraction about 14 s per file and contaminant screening about 8 s per file — noting that absolute times are host-dependent.

## Related tools

- **QC4Metabolomics** (pipeline under test — runs module_File_info, module_TrackCmp and module_Contaminants on each demo mzML file) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (deployment mechanism for the QC4Metabolomics pipeline (docker-compose setup with demo data))
- **R** (runtime of the QC4Metabolomics processing modules being timed)
- **Shiny** (web front-end of the QC4Metabolomics application whose modules are benchmarked)

## Examples

```
docker compose up -d; tail -f qc4metabolomics/log | grep module_Contaminants
```

## Evaluation signals

- Measured quality-parameter extraction time is within the same order of magnitude as the reported ~14 s per demo file.
- Measured contaminant-screening time is within the same order of magnitude as the reported ~8 s per demo file.
- Every demo mzML file has a timing entry in processing_timings.csv (no missing rows).
- Timings are reproducible in direction: extraction step consistently longer than contaminant-screening step on the same file.
- module_Log shows each demo file was actually picked up and processed by module_Files/module_File_schedule before timing began.

## Limitations

- Absolute timings depend on the host machine, so the ~14 s and ~8 s figures are indicative rather than exact.
- Timing is per-file for two modules only (quality-parameter extraction and contaminant screening); it does not capture the full QC4Metabolomics pipeline or dashboard overhead.
- The benchmark reflects the demo data provided with the software, not arbitrary user datasets or instrument raw vendor formats (vendor data must first be converted to mzML).

## Evidence

- [results] On the demo data provided with QC4Metabolomics, extraction of quality parameters takes about 14 s per file and contaminant screening takes an additional about 8 s per file.: [evidence span withheld — anchor status: not_found]
- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths: "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths"
- [methods] module_File_info extracts information (e.g., instrument name, project name, ionization mode and sample name): "module_File_info extracts information (e.g., instrument name, project name, ionization mode and sample name)"
- [methods] module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs: "module_TrackCmp detects peaks based on a user-defined list of m/z and retention time (RT) pairs"
- [methods] module_Contaminants uses a list of nearly 800 known contaminant ions: "module_Contaminants uses a list of nearly 800 known contaminant ions"
- [abstract] QC4Metabolomics is available as code at https://github.com/stanstrup/QC4Metabolomics: "QC4Metabolomics is available as code at https://github.com/stanstrup/QC4Metabolomics"
