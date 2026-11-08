---
name: containerized-pipeline-deployment-with-docker
description: Use when an instrument PC or server must run QC4Metabolomics's continuously
  running file-ingestion process and Shiny web dashboard, especially when R, Shiny,
  renv, xcms, ggplot2/plotly, and mariaDB would otherwise need local installation.
when_to_use_negative:
- When the target environment is a bare instrument PC without Docker but with R/renv
  already set up — the article describes deploying via Docker specifically 'for easy
  deployment', and renv/Docker are the offered options.
- When only a one-off static QC report (e.g. QCScreen or QC-MXP style postanalysis)
  is needed rather than continuously running monitoring — containerization targets
  the two fixed, continuously running processes, not ad-hoc analysis.
- When the underlying software stack (R, Shiny, xcms, mariaDB) cannot be containerized
  due to host restrictions — Docker images and docker-compose are the specific mechanism
  the article provides.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: Docker
  role: Container runtime that packages the QC4Metabolomics two-process setup and
    the ProteoWizard mzML converter so the deployment is OS-agnostic.
- name: QC4Metabolomics
  role: 'The metabolomics QC pipeline being deployed: the continuously running file-ingestion
    modules and the Shiny dashboard containerized via docker-compose.'
  repo: github.com/stanstrup/QC4Metabolomics
- name: ProteoWizard
  role: Runs inside the Docker image to convert vendor-specific raw data to mzML so
    the pipeline stays OS-agnostic.
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:cf8e9f2c2a4cd0b41aa31f2cd316deb2c0a9d576966d0c02430c02dbd962b338
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - ProteoWizard
  - QC4Metabolomics
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# containerized-pipeline-deployment-with-docker

## Summary

Deploy QC4Metabolomics — an R/Shiny quality-control monitor for untargeted MS-based metabolomics — as Docker containers via the provided docker-compose setup, so the two-process architecture (file-ingestion modules + Shiny dashboard) runs identically on any host without local dependency setup.

## When to use

Use when an instrument PC or server must run QC4Metabolomics's continuously running file-ingestion process and Shiny web dashboard, especially when R, Shiny, renv, xcms, ggplot2/plotly, and mariaDB would otherwise need local installation.

## When NOT to use

- When the target environment is a bare instrument PC without Docker but with R/renv already set up — the article describes deploying via Docker specifically 'for easy deployment', and renv/Docker are the offered options.
- When only a one-off static QC report (e.g. QCScreen or QC-MXP style postanalysis) is needed rather than continuously running monitoring — containerization targets the two fixed, continuously running processes, not ad-hoc analysis.
- When the underlying software stack (R, Shiny, xcms, mariaDB) cannot be containerized due to host restrictions — Docker images and docker-compose are the specific mechanism the article provides.

## Inputs

- docker-compose setup file and Docker images from github.com/stanstrup/QC4Metabolomics
- Newly acquired instrument files (mzML, mzXML, mzData, or vendor raw data that the bundled ProteoWizard converter converts to mzML)
- A directory to traverse or a text file containing file paths, per module_Files
- Environment-variable configuration for the containerized services

## Outputs

- Two running processes: the continuously running processing/analysis process and the Shiny web-server dashboard process
- A mariaDB database populated with detected files (module_Files) and extracted metadata/quality parameters (module_File_info, module_TrackCmp, module_Contaminants)
- A web dashboard displaying quality metrics such as observed m/z, retention time, intensity and peak shape for tracked compounds
- Per-file timing of quality-parameter extraction (~14 s) and contaminant screening (~8 s) on demo data

## How to apply

Fetch the Docker images and docker-compose setup file shipped with QC4Metabolomics from github.com/stanstrup/QC4Metabolomics and deploy with docker-compose, since the project provides these 'for easy deployment, along with demo data'. The container image bundles the OS-agnostic ProteoWizard-based vendor-raw-to-mzML converter, so vendor files never need a native instrument conversion step. Configure the deployment through environment variables: point the file watcher (module_Files) at the directory traversed or the text file containing file paths, and configure the mariaDB database and Shiny dashboard connection. Enable the bundled demo data to validate the deployment: module_Files/module_File_schedule should ingest the demo mzML files, module_File_info/module_TrackCmp should extract quality parameters in roughly 14 s per file, and module_Contaminants (screening against nearly 800 known contaminant ions) should take about 8 s per file.

## Related tools

- **Docker** (Container runtime that packages the QC4Metabolomics two-process setup and the ProteoWizard mzML converter so the deployment is OS-agnostic.)
- **QC4Metabolomics** (The metabolomics QC pipeline being deployed: the continuously running file-ingestion modules and the Shiny dashboard containerized via docker-compose.) — github.com/stanstrup/QC4Metabolomics
- **ProteoWizard** (Runs inside the Docker image to convert vendor-specific raw data to mzML so the pipeline stays OS-agnostic.)

## Examples

```
docker-compose up -d  # bring up QC4Metabolomics's processing process, Shiny dashboard, and mariaDB with bundled demo data
```

## Evaluation signals

- The docker-compose deployment brings up both a continuously running processing process and a separate Shiny web-server dashboard process, with a mariaDB database connecting them.
- With demo data enabled, module_Files/module_File_schedule ingest the demo mzML files into module_File_info and module_TrackCmp within the expected per-file timings (extraction ~14 s/file, contaminant screening ~8 s/file).
- Vendor raw data are converted to mzML automatically by the bundled ProteoWizard converter without host-side installation, confirming the image is OS-agnostic.
- The dashboard shows extracted quality metrics (observed m/z, retention time, intensity, peak shape) for the tracked compound list drawn from the containerized database.
- Docker environment-variable configuration correctly wires the file-watcher source (directory or text file of paths) into module_Files and the mariaDB connection.

## Limitations

- Absolute per-file processing times (~14 s extraction, ~8 s contaminant screening) depend on the host machine, so they validate the deployment only as an order-of-magnitude check.
- The demo data validate ingestion, tracking, and contaminant screening but do not validate every module (e.g. module_Warner email alerts, module_Productivity calendar) on real instrument data.
- mzML is the intermediate format — vendor raw data must flow through the ProteoWizard converter step inside the container, adding an upstream conversion dependency.

## Evidence

- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [methods] The system runs as two fixed processes: module_Files watches for new mzML/mzXML/mzData files: [evidence span withheld — anchor status: not_found]
- [methods] An automatic converter that uses ProteoWizard (in Docker and thus OS-agnostic) to convert vendor-specific raw data to mzML: "An automatic converter that uses ProteoWizard
,
 (in Docker and thus OS-agnostic) to convert vendor-specific raw
"
- [results] extraction of quality parameters takes about 14 s per file and contaminant screening takes an additional about 8 s per file.: [evidence span withheld — anchor status: not_found]
- [readme] Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
