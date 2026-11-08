---
name: module-based-software-architecture-mapping
description: 'Use when you need to document or reproduce how a real-time QC pipeline
  is internally structured: which module detects files, which extracts metrics, and
  how the web dashboard consumes them from the database.'
when_to_use_negative:
- The goal is to run QC4Metabolomics on actual metabolomics data rather than to document
  its architecture — use the deployed Shiny app/docker-compose setup instead.
- The target system has a configurable or per-run dynamic pipeline rather than a fixed
  set of modules; this skill's validation assumes a fixed nine-module, two-process
  layout taken verbatim from the paper.
- Input is already a feature table or processed QC metric output — this skill maps
  software architecture upstream of metric extraction, not downstream data analysis.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Subject system whose two-process, nine-module architecture is being mapped
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: Web-server framework hosting the dashboard process of the mapped architecture
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language of the R package containing the nine modules
- name: Docker
  role: Deployment container (with docker-compose) providing the environment-variable
    configuration of the mapped processes; also runs the ProteoWizard mzML converter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:d1da8a2757dbee1a26c5dccb6a4ebce2b924de9df0187a6e28d9397a2d28da24
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# module-based-software-architecture-mapping

## Summary

Reconstructs the fixed two-process modular architecture of QC4Metabolomics — a continuously running processing/analysis process (nine modules) and a Shiny web dashboard, connected by a mariaDB database — so that new mzML/mzXML/mzData instrument files flow into dashboard-visible quality metrics without operator intervention.

## When to use

Use when you need to document or reproduce how a real-time QC pipeline is internally structured: which module detects files, which extracts metrics, and how the web dashboard consumes them from the database.

## When NOT to use

- The goal is to run QC4Metabolomics on actual metabolomics data rather than to document its architecture — use the deployed Shiny app/docker-compose setup instead.
- The target system has a configurable or per-run dynamic pipeline rather than a fixed set of modules; this skill's validation assumes a fixed nine-module, two-process layout taken verbatim from the paper.
- Input is already a feature table or processed QC metric output — this skill maps software architecture upstream of metric extraction, not downstream data analysis.

## Inputs

- Article methods section describing QC4Metabolomics processes and modules
- Repository documentation (http://stanstrup.github.io/QC4Metabolomics/)
- Docker/docker-compose deployment setup and demo data as architectural artifacts

## Outputs

- Architecture specification document (e.g. qc4metabolomics_architecture.md) naming both processes and all nine modules
- Module-to-role mapping (file detection, scheduling, info extraction, peak tracking, contaminant monitoring, alerting, calendar, logging, debug)
- Technology-stack inventory (R, Shiny, xcms, ggplot2/plotly, mariaDB, renv, Docker)

## How to apply

Read the methods section and identify the two independent processes: the continuously running processing/analysis process and the Shiny web-server dashboard process, plus the mariaDB database connecting them. Extract the nine module definitions (module_Files, module_File_schedule, module_File_info, module_TrackCmp, module_Contaminants, module_Warner, module_Productivity, module_Log, module_Debug) and map each to its role in the fixed pipeline: file detection (mzML, mzXML, mzData via directory traversal or a text file of paths), scheduling, filename/sample-info parsing (instrument name, project name, ionization mode, sample name), peak tracking against user-defined m/z and RT pairs, contaminant monitoring against ~800 known contaminant ions, email alerting on threshold exceedance, calendar overview, logging, and debug views. Then extract the upstream data-transfer components (a Windows batch script to copy vendor raw files to network storage and a ProteoWizard-based mzML converter in Docker) and the technology stack (R, Shiny, xcms, ggplot2/plotly, mariaDB, renv, Docker with environment-variable configuration). Write a single architecture specification document (e.g. qc4metabolomics_architecture.md) describing the fixed two-process control loop and the nine modules, generating no per-file runtime artifacts.

## Related tools

- **QC4Metabolomics** (Subject system whose two-process, nine-module architecture is being mapped) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (Web-server framework hosting the dashboard process of the mapped architecture) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language of the R package containing the nine modules)
- **Docker** (Deployment container (with docker-compose) providing the environment-variable configuration of the mapped processes; also runs the ProteoWizard mzML converter)

## Evaluation signals

- The specification document names all nine modules (module_Files, module_File_schedule, module_File_info, module_TrackCmp, module_Contaminants, module_Warner, module_Productivity, module_Log, module_Debug) exactly as listed in the paper.
- The document separates exactly two processes — the continuous processing/analysis process and the Shiny dashboard — connected via mariaDB, matching the methods text.
- Upstream data-flow components (Windows batch copy script and ProteoWizard mzML conversion) are described as feeding the pipeline before module_Files.
- Technology stack mentions match the paper: R, Shiny, xcms, ggplot2/plotly, mariaDB, renv, Docker with environment-variable configuration.
- No per-file runtime artifacts are generated; the deliverable is a single architecture document.

## Limitations

- The architecture is fixed: the paper describes a predefined set of nine modules and two processes, so the mapping does not generalize to user-configurable pipelines.
- The upstream converter only handles vendor raw data convertible via ProteoWizard to mzML; other formats must be pre-converted.
- The public README contains almost no detail (only a link to http://stanstrup.github.io/QC4Metabolomics/), so module-level details must come from the article's methods section.

## Evidence

- [methods] Two fixed processes: "module_Files looks for newly acquired
files (mzML, mzXML, mzData etc.) either by traversing a directory
or by reading from a text file containing file paths and then adds
them to a table in the"
- [methods] Modules: "module_TrackCmp detects peaks based
on a user-defined list of m/z and
retention time (RT) pairs"
- [methods] Contaminants: "module_Contaminants uses
a list of
nearly 800 known contaminant ions"
- [methods] Alerting: "module_Warner can send emails warning
when a quality parameter exceeds a user defined threshold"
- [methods] Upstream converter: "An automatic converter that uses ProteoWizard
,
 (in Docker and thus OS-agnostic) to convert vendor-specific raw
data to mzML"
- [methods] Upstream copy script: "A Windows
batch script that is intended
for use on the instrument PC to copy files to e.g. a network drive
that QC4Metabolomics can access"
- [intro] Web app: "we present QC4Metabolomics, an open-source Shiny web application and R package"
- [abstract] Deployment: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
