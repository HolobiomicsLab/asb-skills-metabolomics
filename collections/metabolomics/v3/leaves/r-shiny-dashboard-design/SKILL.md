---
name: r-shiny-dashboard-design
description: Use when building or adapting a Shiny dashboard that must display instrument
  quality metrics from mzML/mzXML/mzData files as they are acquired (or retrospectively),
  with database-mediated separation between file ingestion and web presentation.
when_to_use_negative:
- Input data are already processed feature tables or corrected intensity drifts —
  this skill covers the acquisition-to-dashboard QC monitoring loop, not postanalysis
  statistical correction workflows like QC-MXP-style drift correction
- The dashboard is for general-purpose data exploration unrelated to instrument QC
  monitoring of pooled-QC and contaminant metrics
- Data are not mass-spectrometry-based metabolomics files (mzML/mzXML/mzData) or cannot
  be converted to them
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Shiny
  role: Web framework for the dashboard process that extracts and displays quality
    metrics
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: R
  role: Implementation language of the QC4Metabolomics package and its nine modules
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Packages the Shiny app and the ProteoWizard-based mzML converter for easy,
    OS-agnostic deployment
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: QC4Metabolomics
  role: Reference implementation of the two-process modular QC dashboard architecture
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:9eb48761e06ff707ab6448f75e7fb9eea971b2b217a9d05424b0dba1fcc8bef6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# r-shiny-dashboard-design

## Summary

Design pattern for an R/Shiny web dashboard that visualizes quality-control metrics (observed m/z, retention time, intensity, peak shape) extracted from untargeted metabolomics MS data in near real time, decoupled from the data-ingestion process via a database. QC4Metabolomics instantiates this pattern as an open-source Shiny web application and R package with nine modular components and Docker deployment.

## When to use

Use when building or adapting a Shiny dashboard that must display instrument quality metrics from mzML/mzXML/mzData files as they are acquired (or retrospectively), with database-mediated separation between file ingestion and web presentation.

## When NOT to use

- Input data are already processed feature tables or corrected intensity drifts — this skill covers the acquisition-to-dashboard QC monitoring loop, not postanalysis statistical correction workflows like QC-MXP-style drift correction
- The dashboard is for general-purpose data exploration unrelated to instrument QC monitoring of pooled-QC and contaminant metrics
- Data are not mass-spectrometry-based metabolomics files (mzML/mzXML/mzData) or cannot be converted to them

## Inputs

- mzML / mzXML / mzData files (converted from vendor raw data via ProteoWizard)
- User-defined list of m/z and retention time (RT) pairs for compound peak tracking
- List of known contaminant ions (~800 preloaded)
- User-defined quality-parameter warning thresholds for email alerting
- Directory path or text file containing file paths to watch for new acquisitions

## Outputs

- Web dashboard views of tracked-compound m/z, retention time, intensity, and peak shape over time
- Contaminant-level monitoring plots (common or user-defined contaminants)
- Email warnings when quality parameters exceed user-defined thresholds
- Calendar overview of instrument productivity (module_Productivity)
- Log of activities of the other modules (module_Log) and technical setup info (module_Debug)
- MariaDB tables populated with file metadata and extracted quality metrics

## How to apply

Implement the system as two independent processes: a continuously running processing process whose modules detect new files (module_Files scans for mzML/mzXML/mzData by traversing a directory or reading a text file of paths), parse filenames and metadata (module_File_schedule, module_File_info), detect peaks against a user-defined list of m/z and retention time (RT) pairs (module_TrackCmp), and check contaminants against a list of nearly 800 known contaminant ions (module_Contaminants); and a separate Shiny web-server dashboard process that extracts and displays the resulting quality metrics. Connect the two processes through a MariaDB table rather than direct calls, so new instrument files flow into the dashboard without operator intervention. Add alerting via module_Warner, which sends emails when a quality parameter exceeds a user-defined threshold, plus a productivity calendar (module_Productivity), activity log (module_Log), and technical debug view (module_Debug). Build in R with Shiny, xcms, and ggplot2/plotly, pin dependencies with renv, and externalize configuration via environment variables in a docker-compose setup so deployment is OS-agnostic.

## Related tools

- **Shiny** (Web framework for the dashboard process that extracts and displays quality metrics) — https://github.com/stanstrup/QC4Metabolomics
- **R** (Implementation language of the QC4Metabolomics package and its nine modules) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Packages the Shiny app and the ProteoWizard-based mzML converter for easy, OS-agnostic deployment) — https://github.com/stanstrup/QC4Metabolomics
- **QC4Metabolomics** (Reference implementation of the two-process modular QC dashboard architecture) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
docker-compose up -d
```

## Evaluation signals

- The architecture separates exactly two processes — the continuously running processing/analysis process and the Shiny web-server dashboard process — connected via MariaDB, with no operator intervention needed for new files to appear on the dashboard
- All nine modules are present and wired in the fixed pipeline: module_Files, module_File_schedule, module_File_info, module_TrackCmp, module_Contaminants, module_Warner, module_Productivity, module_Log, module_Debug
- module_Files successfully picks up newly acquired mzML/mzXML/mzData files by directory traversal or path-list file and adds them to the database table
- module_Warner fires an email only when a quality parameter exceeds the user-defined threshold (no false alerts at in-threshold values)
- Dashboard displays tracked-compound m/z, RT, intensity, and peak-shape trends consistent with the database contents, and deployment via docker-compose reproduces the app with demo data

## Limitations

- Instrument vendors' raw formats must be converted to mzML upstream (via ProteoWizard in Docker); the dashboard does not read vendor-native raw files directly
- Peak tracking requires a user-defined list of m/z and RT pairs, so coverage is limited to compounds the user chooses to track; nonobvious contaminants and ion suppression confined to specific chromatographic regions or particular peaks can escape detection
- Instruments will typically not alert the user or stop acquisition if calibration is inaccurate — e.g., loss of calibration in a limited mass region — so the dashboard's value depends on tracked compounds/contaminants being sensitive to the fault
- Observed RT deviations vary with timescale: less than 0.05 min in the shorter term but at least twice as large between batches, so thresholds must account for batch-to-batch shifts

## Evidence

- [abstract] We therefore introduce QC4Metabolomics , a real-time quality control monitoring software for untargeted metabolomics data.: "We therefore introduce QC4Metabolomics , a real-time quality control monitoring software for untargeted metabolomics data."
- [abstract] and presents the results on a web dashboard: "and presents the results on a web dashboard"
- [intro] an open-source Shiny web application and R package: "an open-source Shiny web application and R package"
- [methods] module_Warner can send emails warning when a quality parameter exceeds a user defined threshold: "module_Warner can send emails warning when a quality parameter exceeds a user defined threshold"
- [methods] module_Productivity provides a calendar overview: "module_Productivity
provides a calendar
overview"
- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
- [readme] QC systems for metabolomics studies: "QC systems for metabolomics studies"
