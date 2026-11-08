---
name: spectral-library-based-feature-extraction
description: Use when you need feature extraction and internal-standard identification
  from raw vendor LC-MS/MS files during an ongoing acquisition — e.g., Thermo Orbitrap
  RAW files converted to mzML and matched against an .
when_to_use_negative:
- The input is already a processed feature table or peak list rather than raw vendor
  LC-MS/MS data requiring MSConvert/MS-DIAL processing.
- The goal is targeted quantitation of non-internal-standard analytes with a full
  spectral library outside the scope of instrument-run QC, rather than rapid per-file
  QC during acquisition.
- You are working with acquisition sequences or data formats from vendors other than
  Thermo (Agilent, Bruker, Sciex, Waters), since Rapid QC-MS has only been tested
  extensively on Thermo RAW files.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Rapid QC-MS
  role: 'Orchestrates the QC job: watches for new raw files, feeds them through MSConvert
    and MS-DIAL, and aggregates extracted internal-standard features into a Pass/Fail/Warn
    dashboard.'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts raw vendor data (Thermo RAW) into open mzML format before routing
    it to the MS-DIAL feature-extraction pipeline.
- name: MS-DIAL
  role: Performs spectral-library-based feature extraction and identification against
    the .msp library on the mzML files.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:df2fe05bd21147e66beac433d84c9819a12f3232f5d9ceb747bb0793ef91e9cb
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
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Spectral library-based feature extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts and identifies metabolite features from LC-MS/MS mzML files by matching against a spectral library (.msp) via MS-DIAL, as the data-processing stage inside Rapid QC-MS that yields internal-standard retention time, m/z, and intensity metrics for real-time QC.

## When to use

Use when you need feature extraction and internal-standard identification from raw vendor LC-MS/MS files during an ongoing acquisition — e.g., Thermo Orbitrap RAW files converted to mzML and matched against an .msp library so per-specimen quality-control metrics appear on the dashboard within minutes.

## When NOT to use

- The input is already a processed feature table or peak list rather than raw vendor LC-MS/MS data requiring MSConvert/MS-DIAL processing.
- The goal is targeted quantitation of non-internal-standard analytes with a full spectral library outside the scope of instrument-run QC, rather than rapid per-file QC during acquisition.
- You are working with acquisition sequences or data formats from vendors other than Thermo (Agilent, Bruker, Sciex, Waters), since Rapid QC-MS has only been tested extensively on Thermo RAW files.

## Inputs

- Raw vendor LC-MS/MS data files (e.g., Thermo .raw)
- Converted open-format mzML files
- Spectral library in .msp format (e.g., internal standard library from Zenodo 10.5281/zenodo.10525183)
- Acquisition sequence .csv defining the run order

## Outputs

- Per-specimen internal standard feature table with retention time, m/z, and intensity
- Pass/Fail/Warn QC designation for each internal standard target within each specimen
- Updated browser-viewable dashboard of quality-control metrics (sensitivity, RT shifts, mass accuracy drift)

## How to apply

Convert each newly written raw file (e.g., Thermo RAW) into open mzML format with MSConvert 3.0.23332-7faa7f422 before routing it to the data-processing pipeline. Then run MS-DIAL ver.4.9.221218 on the mzML to perform feature extraction against a curated spectral library in .msp format, matching precursor m/z and retention time of internal standards. Rapid QC-MS orchestrates this automatically: after installation (`py -m pip install rapid-qc-ms`, which pulls Pandas, SQLAlchemy, Plotly Dash, Bootstrap, Watchdog, Google API and Slack API but requires manual installation of MSConvert and MS-DIAL), launch with `rapidqcms` and configure a new QC job pointing at the raw files, the acquisition sequence .csv, and the .msp library. The rationale is to continuously evaluate instrument sensitivity, in-run retention time shifts, and mass accuracy drift by quantifying each internal standard's observed RT, m/z, and intensity, so deviations in these extracted features trigger Pass/Fail/Warn designations and optional Slack or email alerts. Because MSConvert normalizes all vendor formats into mzML, the extracted features are vendor-agnostic, though the pipeline has only been tested extensively on Thermo acquisition sequences and RAW files.

## Related tools

- **Rapid QC-MS** (Orchestrates the QC job: watches for new raw files, feeds them through MSConvert and MS-DIAL, and aggregates extracted internal-standard features into a Pass/Fail/Warn dashboard.) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts raw vendor data (Thermo RAW) into open mzML format before routing it to the MS-DIAL feature-extraction pipeline.)
- **MS-DIAL** (Performs spectral-library-based feature extraction and identification against the .msp library on the mzML files.)

## Examples

```
py -m pip install rapid-qc-ms; rapidqcms
```

## Evaluation signals

- Each internal standard target within a specimen receives a Pass, Fail, or Warn designation (missing IS are marked Fail), matching the criteria described in the specimen table.
- Extracted retention time, m/z, and intensity values for every internal standard appear on the interactive dashboard within minutes of the raw file being written.
- Running the pipeline over the Zenodo example files (sequence .csv, library .msp, and .raw files) regenerates the per-specimen, per-IS QC assignments reported in the paper.
- MSConvert output is valid mzML for every input raw file, and MS-DIAL successfully matches all library entries defined in the .msp file.
- Newly acquired files that fail internal-standard criteria trigger Slack or email notifications, confirming the feature-extraction-based QC logic is active.

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may contain bugs.
- MSConvert and MS-DIAL must be installed manually; they are not installed automatically by the rapid-qc-ms package.
- The pipeline was designed to run on Windows platforms because of its dependency on MSConvert and MS-DIAL; MacOS users can only monitor/view run data.
- Installation of Python and Python packages on MS instrument computers carries some risk, though a virtual environment can mitigate it.

## Evidence

- [full_text] MS-DIAL performs feature extraction against the .msp library as part of the Rapid QC-MS pipeline: [evidence span withheld — anchor status: not_found]
- [results] Per-specimen internal standard pass/fail/warn assignments are the direct output of the library-based feature extraction: [evidence span withheld — anchor status: not_found]
