---
name: lc-ms-ms-quality-control-job-configuration
description: Use when Thermo Orbitrap .raw files are being written during an active
  LC-MS/MS run and you need near-real-time QC (instrument sensitivity, retention time
  shifts, mass accuracy) against a sequence .csv and library .msp.
when_to_use_negative:
- You need fully validated processing of Agilent, Bruker, Sciex, or Waters acquisition
  sequences or data files — Rapid QC-MS has only been tested extensively on Thermo
- Input data are already converted mzML or already a quantified/annotated feature
  table, so no job watching for new vendor raw files is needed
- You are running on macOS and need the full conversion pipeline — MSConvert/MS-DIAL
  dependencies mean Rapid QC-MS on MacOS supports monitoring/viewing run data only
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Rapid QC-MS
  role: Host application that starts and configures the QC job, watches for new raw
    files, and presents the dashboard
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts each completed Thermo raw file into open mzML format before routing
    to the data processing pipeline
- name: MS-DIAL
  role: Processes the mzML files, performing quantification and annotation of all
    user-defined targets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:0624219f135a1b75df83a70860b0a5a4ad382639da0aac829927e8477043f65a
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
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lc-ms-ms-quality-control-job-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and run a Rapid QC-MS job that watches a Thermo acquisition for newly written .raw files, converts each via MSConvert to mzML, quantifies/annotates user-defined targets with MS-DIAL, and updates a browser dashboard with LC-MS/MS quality metrics.

## When to use

Use when Thermo Orbitrap .raw files are being written during an active LC-MS/MS run and you need near-real-time QC (instrument sensitivity, retention time shifts, mass accuracy) against a sequence .csv and library .msp.

## When NOT to use

- You need fully validated processing of Agilent, Bruker, Sciex, or Waters acquisition sequences or data files — Rapid QC-MS has only been tested extensively on Thermo
- Input data are already converted mzML or already a quantified/annotated feature table, so no job watching for new vendor raw files is needed
- You are running on macOS and need the full conversion pipeline — MSConvert/MS-DIAL dependencies mean Rapid QC-MS on MacOS supports monitoring/viewing run data only

## Inputs

- Thermo Fisher .raw files written during an active acquisition sequence
- Acquisition sequence file (.csv)
- Target spectral library (.msp)
- Optionally, example data from Metabolomics Workbench (DOI 10.21228/M8Z119)

## Outputs

- Converted mzML files per completed raw file
- MS-DIAL quantification/annotation results for user-defined targets
- Browser-viewable QC dashboard with metrics (instrument sensitivity, retention time shifts, mass accuracy drift)
- QC fail notifications via Slack or email

## How to apply

Start Rapid QC-MS from the command line with 'rapidqcms' and create a new QC job supplying the Thermo acquisition sequence .csv, the target library .msp, and the directory where the instrument writes .raw files (example files from Zenodo 10.5281/zenodo.10525183 and Metabolomics Workbench DOI 10.21228/M8Z119 can be used for setup). Configure the job to watch for completed raw files; each completed Thermo .raw is converted to open mzML using MSConvert (version 3.0.23332-7faa7f422) so the pipeline works across vendor formats. The mzML is routed to MS-DIAL (ver.4.9.22121823), which quantifies and annotates all user-defined targets. Rapid QC-MS then evaluates the resulting metrics — instrument sensitivity, in-run retention time shifts, and mass accuracy drift — updating a browser-viewable dashboard minutes after each file is written and alerting users (e.g., Slack/email) when a newly acquired raw file fails specified requirements.

## Related tools

- **Rapid QC-MS** (Host application that starts and configures the QC job, watches for new raw files, and presents the dashboard) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts each completed Thermo raw file into open mzML format before routing to the data processing pipeline)
- **MS-DIAL** (Processes the mzML files, performing quantification and annotation of all user-defined targets)

## Examples

```
rapidqcms
```

## Evaluation signals

- Confirm a valid mzML file was produced in the job's output directory for each completed Thermo .raw
- Confirm MS-DIAL produced quantification/annotation results for every user-defined target in the .msp library
- Verify the dashboard updates with QC results within minutes of each new data file being written
- Check that reported metrics (instrument sensitivity, in-run retention time shifts, mass accuracy) are populated and consistent across samples
- Verify alerts are triggered only for newly acquired raw files that do not meet the specified requirements

## Limitations

- MSConvert and MS-DIAL must be downloaded and installed manually; they are not installed by the rapid-qc-ms pip package
- Rapid QC-MS was designed for Windows platforms because of its MSConvert and MS-DIAL dependencies; MacOS users can only monitor/view instrument run data
- Only tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs
- Requires Python 3.8–3.11

## Evidence

- [readme] example inputs for job setup: [evidence span withheld — anchor status: not_found]
- [readme] platform limitation: [evidence span withheld — anchor status: not_found]
