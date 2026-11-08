---
name: docker-container-deployment
description: Use when deploying QC4Metabolomics or its Shiny/R modules (module_Files,
  module_TrackCmp, module_Contaminants, etc.) on a server or workstation, or when
  a reproducible demo/test instance of the QC pipeline is needed for mzML-based untargeted
  metabolomics QC.
when_to_use_negative:
- You only need a single QC report for an already-complete acquisition — QC4Metabolomics
  is designed for continuous real-time or retrospective monitoring rather than one-off
  analysis
- Your data are not MS-based untargeted metabolomics files (mzML/mzXML/mzData); the
  pipeline's converters and QC views target that data type
- The host environment cannot run Docker or you need to embed QC as a library call
  inside an existing R script rather than run a standalone service
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: The Shiny web application and R package whose containerized deployment this
    skill covers; monitors files as they are acquired or retrospectively by tracking
    user-defined compounds and presenting results on a web dashboard
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Provided as Docker images and a docker-compose setup file for easy, reproducible
    deployment of the QC stack
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Shiny
  role: The web framework of the deployed dashboard container
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Language of the QC4Metabolomics package running inside the container
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:44f6801b1d3efb731ae47ac4bb3d39958f23ebf520b8a71742285ad6741d268f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# docker-container-deployment

## Summary

Deploy QC4Metabolomics, an open-source Shiny web application and R package for real-time and retrospective QC of untargeted MS-based metabolomics data, as a reproducible containerized stack using the provided Docker images and docker-compose setup, including demo data for validation.

## When to use

Use when deploying QC4Metabolomics or its Shiny/R modules (module_Files, module_TrackCmp, module_Contaminants, etc.) on a server or workstation, or when a reproducible demo/test instance of the QC pipeline is needed for mzML-based untargeted metabolomics QC.

## When NOT to use

- You only need a single QC report for an already-complete acquisition — QC4Metabolomics is designed for continuous real-time or retrospective monitoring rather than one-off analysis
- Your data are not MS-based untargeted metabolomics files (mzML/mzXML/mzData); the pipeline's converters and QC views target that data type
- The host environment cannot run Docker or you need to embed QC as a library call inside an existing R script rather than run a standalone service

## Inputs

- docker-compose setup file and Docker images from the QC4Metabolomics repository
- Vendor-specific raw metabolomics files or mzML/mzXML/mzData files to be monitored
- Optional user-defined contaminant ion list (m/z and retention time pairs) or the bundled list of nearly 800 known contaminant ions
- Bundled demo data shipped with the compose setup for validation

## Outputs

- A running multi-container stack (ProteoWizard converter, mariaDB, Shiny web dashboard) accessible via browser
- mariaDB database tables storing per-file diagnostic information (observed m/z, retention time, intensity, peak shape) and contaminant detections
- A populated Shiny dashboard rendering QC views (e.g. contaminant time view, sample-composition barplot, heatmap) for the demo or user data

## How to apply

Obtain the docker-compose setup file and Docker images from the QC4Metabolomics repository (github.com/stanstrup/QC4Metabolomics) and start the full stack with docker-compose. The stack includes an OS-agnostic ProteoWizard converter container that translates vendor-specific raw files to mzML, a mariaDB database that stores extracted diagnostics (observed m/z, retention time, intensity, peak shape), and the Shiny dashboard container. Configure modules via environment variables passed to the container (e.g. to enable module_Contaminants with the bundled ~800 known contaminant ions or a user-supplied list). Use the bundled demo data to smoke-test the deployment: module_Files should detect the demo mzML files, module_File_schedule should queue them, and the dashboard should populate with QC views. Rationale: the article states Docker images and a docker-compose setup file are provided specifically 'for easy deployment', so containerizing avoids manual dependency management of the R package, Shiny app, and database.

## Related tools

- **QC4Metabolomics** (The Shiny web application and R package whose containerized deployment this skill covers; monitors files as they are acquired or retrospectively by tracking user-defined compounds and presenting results on a web dashboard) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Provided as Docker images and a docker-compose setup file for easy, reproducible deployment of the QC stack) — https://github.com/stanstrup/QC4Metabolomics
- **Shiny** (The web framework of the deployed dashboard container) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Language of the QC4Metabolomics package running inside the container) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
git clone https://github.com/stanstrup/QC4Metabolomics && cd QC4Metabolomics && docker-compose up -d
```

## Evaluation signals

- docker-compose up brings up all expected services (ProteoWizard converter, mariaDB, Shiny) without errors
- With demo data, module_Files detects the demo mzML files and module_File_schedule queues them for processing
- The Shiny dashboard renders QC views (e.g. contaminant time view, sample-composition barplot, heatmap) populated with demo-data values
- Diagnostic records in mariaDB contain observed m/z, retention time, intensity and peak shape per file
- module_Contaminants, when enabled via environment variables, detects contaminant peaks matching entries in the ~800-ion list

## Limitations

- Deployment assumes the instrument PC can copy raw files to a network drive accessible to the containers (the article describes a Windows batch script for this purpose)
- Environment-variable configuration is required to enable optional modules such as module_Contaminants and to supply contaminant lists
- The README itself is minimal — detailed documentation lives at the project's documentation site (stanstrup.github.io/QC4Metabolomics), so container setup details beyond the compose file must be sourced there

## Evidence

- [abstract] Docker images and docker-compose for easy deployment: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [methods] OS-agnostic ProteoWizard converter in Docker: [evidence span withheld — anchor status: not_found]
- [methods] Environment variables configure module_Contaminants: "module_Contaminants uses a list of nearly 800 known contaminant ions"
- [abstract] Demo data available for testing: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
