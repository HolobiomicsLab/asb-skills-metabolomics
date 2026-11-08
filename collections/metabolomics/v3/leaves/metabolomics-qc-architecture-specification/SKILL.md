---
name: metabolomics-qc-architecture-specification
description: Use when you need to specify, reproduce, or modify the internal control
  flow of QC4Metabolomics — e.g. mapping how module_Files detects new files, how modules
  hand off through mariaDB, and how the Shiny dashboard displays quality metrics.
when_to_use_negative:
- The goal is to run QC monitoring on actual mzML data rather than to document or
  reconstruct the system's internal architecture
- The task concerns postanalysis drift correction or QC metric interpretation (e.g.
  RT/m/z deviation plots) rather than the software control flow
- You need per-file runtime processing artifacts — this skill explicitly produces
  a single fixed architecture document with no per-file outputs
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: QC4Metabolomics
  role: The system whose two-process modular architecture (processing pipeline + Shiny
    dashboard) is being specified
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web-server dashboard process that extracts and displays quality metrics from
    the mariaDB database
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Deployment container (docker-compose setup) including the OS-agnostic ProteoWizard
    mzML converter and environment-variable configuration
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language of the R package and the nine processing modules
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:90058e6b8d05b4a4cd8a4b0c06f829e4bcb2393c50d24694d320bed53d6bb947
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# metabolomics-qc-architecture-specification

## Summary

A skill for reconstructing and documenting the fixed two-process architecture of QC4Metabolomics: a continuously running processing/analysis pipeline (nine R modules plus a mariaDB database) feeding a Shiny web dashboard, so that newly acquired mzML/mzXML/mzData files flow into quality metrics without operator intervention.

## When to use

Use when you need to specify, reproduce, or modify the internal control flow of QC4Metabolomics — e.g. mapping how module_Files detects new files, how modules hand off through mariaDB, and how the Shiny dashboard displays quality metrics.

## When NOT to use

- The goal is to run QC monitoring on actual mzML data rather than to document or reconstruct the system's internal architecture
- The task concerns postanalysis drift correction or QC metric interpretation (e.g. RT/m/z deviation plots) rather than the software control flow
- You need per-file runtime processing artifacts — this skill explicitly produces a single fixed architecture document with no per-file outputs

## Inputs

- QC4Metabolomics methods section / paper text describing the two processes and nine modules
- Repository README and documentation at http://stanstrup.github.io/QC4Metabolomics/
- Module definitions and roles (module_Files through module_Debug) from the article
- Docker/docker-compose deployment configuration referenced in the abstract

## Outputs

- qc4metabolomics_architecture.md — a single architecture specification document naming both processes, the mariaDB connection, all nine modules, and the auxiliary data-transfer components

## How to apply

Read the methods section and identify the two independent processes: the continuously running processing/analysis process and the Shiny web-server dashboard process, connected by the mariaDB database. Enumerate the nine modules in their fixed pipeline roles: module_Files (detecting new mzML/mzXML/mzData files by traversing a directory or reading a text file of paths and adding them to a database table), module_File_schedule (checking for new filenames in the database), module_File_info (extracting instrument name, project name, ionization mode, sample name), module_TrackCmp (peak detection on user-defined m/z and RT pairs), module_Contaminants (~800 known contaminant ions), module_Warner (email alerts when a quality parameter exceeds a user-defined threshold), module_Productivity (calendar overview), module_Log, and module_Debug. Include the upstream data-transfer components — a Windows batch script that copies files from the instrument PC to a network drive and a ProteoWizard-based mzML converter running in Docker — and the technology stack (R, Shiny, xcms, ggplot2/plotly, mariaDB, renv, Docker with environment-variable configuration). Write a single architecture specification document (e.g. qc4metabolomics_architecture.md) describing the fixed two-process control loop; no per-file runtime artifacts are generated.

## Related tools

- **QC4Metabolomics** (The system whose two-process modular architecture (processing pipeline + Shiny dashboard) is being specified) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web-server dashboard process that extracts and displays quality metrics from the mariaDB database) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Deployment container (docker-compose setup) including the OS-agnostic ProteoWizard mzML converter and environment-variable configuration) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language of the R package and the nine processing modules) — https://github.com/stanstrup/QC4Metabolomics

## Evaluation signals

- The specification document names all nine modules (module_Files, module_File_schedule, module_File_info, module_TrackCmp, module_Contaminants, module_Warner, module_Productivity, module_Log, module_Debug) exactly as in the paper
- Both processes are described with their separation made explicit: the continuously running processing/analysis process vs. the Shiny web-server dashboard process
- module_Files is described as accepting new files 'either by traversing a directory or by reading from a text file containing file paths' and adding them to a database table
- The auxiliary upstream components (Windows batch script for copying from the instrument PC, ProteoWizard mzML converter in Docker) are included
- Exactly one architecture document is produced; module counts and process separation match the methods section text, with no per-file runtime artifacts

## Limitations

- The architecture is fixed: modules play predetermined roles in the pipeline; the skill documents this fixed design rather than a configurable or extensible pipeline
- The repository README is minimal and defers all documentation to http://stanstrup.github.io/QC4Metabolomics/, so the paper's methods section is the primary source for module definitions
- Instruments typically do not alert the user or stop acquisition when calibration is inaccurate — the architecture's alerting (module_Warner) exists precisely to compensate, but only for user-defined thresholds

## Evidence

- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table in the: "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths"
- [methods] module_Warner can send emails warning when a quality parameter exceeds a user defined threshold: "module_Warner can send emails warning when a quality parameter exceeds a user defined threshold"
- [methods] A Windows batch script that is intended for use on the instrument PC to copy files to e.g. a network drive that QC4Metabolomics can access: "A Windows batch script that is intended for use on the instrument PC to copy files to e.g. a network drive that QC4Metabolomics can access"
- [methods] An automatic converter that uses ProteoWizard, (in Docker and thus OS-agnostic) to convert vendor-specific raw data to mzML: [evidence span withheld — anchor status: not_found]
- [abstract] QC4Metabolomics monitors files as they are acquired or retrospectively by tracking any user-defined compound(s): "QC4Metabolomics monitors files as they are acquired or retrospectively by tracking any user-defined compound(s)"
- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [readme] README: QC systems for metabolomics studies. Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
