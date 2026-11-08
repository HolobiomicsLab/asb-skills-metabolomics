---
name: lcms-data-quality-control-assessment
description: Use when running or monitoring large LC-MS/MS untargeted metabolomics
  analyses (e.g. Thermo Orbitrap acquisition sequences) and you need per-specimen
  QC status (Pass/Fail/Warn) for internal standards within minutes of each raw file
  being written.
when_to_use_negative:
- The data are from Agilent, Bruker, Sciex, or Waters acquisition sequences and vendor
  formats — the tool has only been tested extensively on Thermo Fisher mass spectrometers,
  Thermo acquisition sequences, and Thermo RAW files.
- The analysis is post-hoc batch QC with no need for real-time, in-run monitoring
  — Rapid QC-MS is designed for synchronous assessment during data acquisition, not
  retrospective reprocessing.
- The environment is MacOS-only — MSConvert and MS-DIAL dependencies are Windows-oriented,
  and MacOS users can only use Rapid QC-MS to monitor/view instrument run data, not
  run the full QC pipeline.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: Rapid QC-MS
  role: 'Core tool that orchestrates the QC job: watches for new raw files, invokes
    MSConvert and MS-DIAL, evaluates IS/RS/ES pass/fail/warn criteria, and renders
    the interactive Dash dashboard with Slack/email alerts.'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts raw acquired Thermo data into open mzML format before routing it
    to the data-processing pipeline, enabling vendor-agnostic downstream QC.
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: MS-DIAL
  role: Performs feature extraction and identification against the .msp library on
    the mzML output from MSConvert, supplying the internal standard metrics that Rapid
    QC-MS evaluates.
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:26b056c170b96600733e03f45b4323ec0f5ce6cfa34df2a3a3b8196a870211ef
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# lcms-data-quality-control-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated, real-time LC-MS/MS data quality assessment using Rapid QC-MS, which converts Thermo RAW files to mzML via MSConvert, processes them with MS-DIAL against an .msp library, and surfaces internal standard retention time, m/z, and intensity QC results on a browser dashboard with Slack/email alerts.

## When to use

Use when running or monitoring large LC-MS/MS untargeted metabolomics analyses (e.g. Thermo Orbitrap acquisition sequences) and you need per-specimen QC status (Pass/Fail/Warn) for internal standards within minutes of each raw file being written.

## When NOT to use

- The data are from Agilent, Bruker, Sciex, or Waters acquisition sequences and vendor formats — the tool has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files.
- The analysis is post-hoc batch QC with no need for real-time, in-run monitoring — Rapid QC-MS is designed for synchronous assessment during data acquisition, not retrospective reprocessing.
- The environment is MacOS-only — MSConvert and MS-DIAL dependencies are Windows-oriented, and MacOS users can only use Rapid QC-MS to monitor/view instrument run data, not run the full QC pipeline.

## Inputs

- Thermo RAW files (vendor-format LC-MS/MS acquisition data)
- Thermo acquisition sequence .csv file
- .msp spectral library defining internal standards (IS), reference standards (RS), and external standards (ES)
- MSConvert 3.0.23332-7faa7f422 installation
- MS-DIAL ver.4.9.221218 installation

## Outputs

- Per-specimen, per-internal-standard Pass/Fail/Warn QC assignments (missing IS marked Fail)
- Browser-viewable interactive dashboard updated minutes after each new data file is written, with retention time, m/z, and intensity visualizations
- Slack or email alert notifications on QC fails
- Google Drive–synced QC results accessible remotely with Google-authenticated access

## How to apply

Install Rapid QC-MS with `py -m pip install rapid-qc-ms` on a Windows machine running Python 3.8–3.11; manually install MSConvert 3.0.23332-7faa7f422 and MS-DIAL ver.4.9.221218 since the Python dependencies (pandas, SQLAlchemy, Plotly Dash, Bootstrap, Watchdog, Google API, Slack API) are installed automatically but these MS tools are not. Configure a new QC job pointing at the acquired Thermo RAW files, the Thermo acquisition sequence .csv, and an .msp library of internal standards (IS), reference standards (RS), or external standards (ES); optionally sync to Google Drive for remote monitoring. Launch the dashboard with `rapidqcms` and let MSConvert convert each raw vendor file into open mzML format before routing it to the MS-DIAL data-processing pipeline, which extracts features against the .msp library. Rapid QC-MS then evaluates IS, RS, and ES together to assign each internal standard target within a specimen a Pass, Fail, or Warn designation (missing IS are marked Fail) and updates the browser-viewable dashboard minutes after the new data file is written, spanning metrics such as instrument sensitivity, in-run retention time shifts, and mass accuracy drift.

## Related tools

- **Rapid QC-MS** (Core tool that orchestrates the QC job: watches for new raw files, invokes MSConvert and MS-DIAL, evaluates IS/RS/ES pass/fail/warn criteria, and renders the interactive Dash dashboard with Slack/email alerts.) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts raw acquired Thermo data into open mzML format before routing it to the data-processing pipeline, enabling vendor-agnostic downstream QC.) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **MS-DIAL** (Performs feature extraction and identification against the .msp library on the mzML output from MSConvert, supplying the internal standard metrics that Rapid QC-MS evaluates.) — http://prime.psc.riken.jp/compms/msdial/main.html

## Examples

```
rapidqcms
```

## Evaluation signals

- Confirm that each internal standard target within each specimen receives exactly one Pass/Fail/Warn designation, and that missing IS are marked Fail.
- Verify that the dashboard updates within minutes of a new raw file being written, matching the paper's claim of synchronous QC during acquisition.
- Check that regenerated per-specimen IS assignments on the Zenodo example dataset (10.5281/zenodo.10525183) reproduce the determinations reported in the paper's Specimen Table.
- Ensure Slack or email alert notifications fire whenever a newly acquired raw file does not meet specified requirements.
- Validate that MSConvert output is valid mzML before it reaches MS-DIAL, and that feature extraction against the .msp library yields retention time, m/z, and intensity values for all configured IS.

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats are expected to contain bugs.
- MSConvert and MS-DIAL must be installed manually — the Python pip install does not handle them.
- MacOS users can only monitor/view instrument run data; the full automated QC pipeline requires a Windows environment.
- There is no changelog documented in the article's discussion, so version-specific behavioral changes are not traceable from the manuscript alone.

## Evidence
