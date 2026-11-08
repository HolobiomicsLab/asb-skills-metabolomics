---
name: thermo-acquisition-sequence-configuration
description: 'Use when running Rapid QC-MS quality control over a Thermo Orbitrap
  LC-MS/MS run: you have a Thermo acquisition sequence .csv, Thermo .raw files, and
  a spectral library .msp and need automated, specimen-level QC assignments minutes
  after each file is written.'
when_to_use_negative:
- The acquisition sequence or raw files are from Agilent, Bruker, Sciex, or Waters
  — Rapid QC-MS has only been tested extensively on Thermo acquisition sequences and
  Thermo RAW files and may have vendor-format bugs.
- Data are already converted feature tables or processed peak lists — this skill assumes
  vendor RAW input is available for MSConvert-to-mzML conversion and MS-DIAL processing.
- The platform is not Windows and vendor RAW conversion is required — Rapid QC-MS
  depends on MSConvert and MS-DIAL and was designed for Windows; MacOS users can only
  monitor/view run data.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Rapid QC-MS
  role: 'Core skill software: launches the QC job, monitors the Thermo acquisition
    sequence, assigns per-specimen/per-IS Pass/Fail/Warn statuses, and renders the
    live dashboard.'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts each newly written Thermo RAW file into open mzML format before routing
    it to the Rapid QC-MS data-processing pipeline.
- name: MS-DIAL
  role: Performs feature extraction and identification against the .msp library on
    the mzML files produced by MSConvert.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:4aeca12a9db24a0a2a776bb5ebcb55265b6850576630f142e6327a1a8e7c31cc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  repo_url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - MS-DIAL
  - MSConvert
  - MSConvert 3.0.23332-7faa7f422
  - Rapid QC-MS
  - Thermo Orbitrap
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# thermo-acquisition-sequence-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rapid QC-MS is configured against a Thermo acquisition sequence .csv so that each newly written Thermo RAW file is automatically converted to mzML, processed via MS-DIAL against a .msp library, and scored per specimen and per internal standard (Pass/Fail/Warn) on a live dashboard.

## When to use

Use when running Rapid QC-MS quality control over a Thermo Orbitrap LC-MS/MS run: you have a Thermo acquisition sequence .csv, Thermo .raw files, and a spectral library .msp and need automated, specimen-level QC assignments minutes after each file is written.

## When NOT to use

- The acquisition sequence or raw files are from Agilent, Bruker, Sciex, or Waters — Rapid QC-MS has only been tested extensively on Thermo acquisition sequences and Thermo RAW files and may have vendor-format bugs.
- Data are already converted feature tables or processed peak lists — this skill assumes vendor RAW input is available for MSConvert-to-mzML conversion and MS-DIAL processing.
- The platform is not Windows and vendor RAW conversion is required — Rapid QC-MS depends on MSConvert and MS-DIAL and was designed for Windows; MacOS users can only monitor/view run data.

## Inputs

- Thermo acquisition sequence .csv
- Thermo RAW files (.raw)
- Spectral library .msp (internal standards)
- Vendor .raw converted to mzML by MSConvert

## Outputs

- Per-specimen, per-internal-standard Pass/Fail/Warn QC designations
- Browser-viewable interactive dashboard of IS retention time, m/z, and intensity across samples
- Optional Google Drive-synced QC results with Google-authenticated remote access
- Slack or email notifications on QC fails

## How to apply

Install Rapid QC-MS with `py -m pip install rapid-qc-ms` (Python 3.8–3.11; pandas, SQLAlchemy, Plotly Dash, Bootstrap, Watchdog, Google API and Slack API dependencies install automatically), then manually install MSConvert 3.0.23332–7faa7f422 and MS-DIAL ver.4.9.22121823. Launch the tool by entering `rapidqcms` in the terminal and, following the quickstart guide, create a new QC job that points at the Thermo acquisition sequence .csv, the vendor .raw files, and the internal-standard spectral library .msp (example files: sequence .csv, library .msp, and additional .raw files at Zenodo 10.5281/zenodo.10525183; example data DOI 10.21228/M8Z119 on Metabolomics Workbench). As each Thermo RAW file is written, MSConvert converts it into open mzML format before routing it to the data-processing pipeline, where MS-DIAL performs feature extraction and identification against the .msp library. Rapid QC-MS then evaluates instrument sensitivity, in-run retention-time shifts, and mass-accuracy drift across internal standards, reference standards, and expected specimens, assigns each internal standard within each specimen a Pass, Fail, or Warn designation (missing IS are marked Fail), and updates a browser-viewable dashboard minutes after the new data file is written. Optionally enable remote QC by syncing results to Google Drive and receive Slack or email notifications whenever a newly acquired raw file does not meet specified requirements.

## Related tools

- **Rapid QC-MS** (Core skill software: launches the QC job, monitors the Thermo acquisition sequence, assigns per-specimen/per-IS Pass/Fail/Warn statuses, and renders the live dashboard.) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts each newly written Thermo RAW file into open mzML format before routing it to the Rapid QC-MS data-processing pipeline.)
- **MS-DIAL** (Performs feature extraction and identification against the .msp library on the mzML files produced by MSConvert.)

## Examples

```
py -m pip install rapid-qc-ms && rapidqcms
```

## Evaluation signals

- Every internal standard target within each specimen receives a Pass, Fail, or Warn designation, with missing IS explicitly marked Fail.
- Regenerated per-specimen IS assignments on the Zenodo example files (sequence .csv, library .msp, additional .raw files) reproduce the pass/fail/warn determinations reported in the paper's specimen table.
- The browser-viewable dashboard updates within minutes after a new data file is written, covering instrument sensitivity, in-run retention-time shifts, and mass-accuracy drift.
- Slack or email notifications fire for specimens with potentially poor analytical integrity (raw files not meeting specified requirements).
- MSConvert output exists as mzML for every processed Thermo RAW file before MS-DIAL feature extraction runs.

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have processing bugs and need community support.
- MSConvert and MS-DIAL must be installed manually; they are not installed automatically by `py -m pip install rapid-qc-ms`.
- The software was designed for Windows; MacOS users can monitor/view run data but the full pipeline relies on MSConvert and MS-DIAL.
- Only Python 3.8–3.11 is supported.

## Evidence
