---
name: thermo-instrument-data-handling
description: Use when a completed Thermo .raw file is written during an active instrument
  run and must be converted to mzML (MSConvert) then quantified/annotated (MS-DIAL)
  for automated QC.
when_to_use_negative:
- Processing raw files from non-Thermo vendors (Agilent, Bruker, Sciex, Waters) —
  Rapid QC-MS has only been tested extensively on Thermo acquisition sequences and
  Thermo RAW files, and bugs are expected with other vendor formats
- Running the conversion/processing stage on MacOS — MSConvert and MS-DIAL dependencies
  mean MacOS users can only monitor/view instrument run data, not process files
- Input data is already in mzML or a processed feature/quantification table — no raw
  vendor conversion stage is needed
tools:
- name: Rapid QC-MS
  role: 'Orchestrates the job: watches for newly written raw files, invokes MSConvert,
    routes mzML to MS-DIAL, and writes results to the output directory'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts each completed Thermo .raw file into open mzML format before routing
    it to the data processing pipeline
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: MS-DIAL ver.4.9.22121823
  role: Processes the converted mzML files, carrying out quantification and annotation
    for all user-defined targets
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:8d026931666bb8cb36e53f561f9550628b8b74619b5dd259bbf72d878fec36a4
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
  - MS-DIAL ver.4.9.22121823
  - MSConvert
  - MSConvert 3.0.23332-7faa7f422
  - Rapid QC-MS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# thermo-instrument-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Handles Thermo .raw files generated during an active LC-MS/MS acquisition by watching for newly written files, converting them to open mzML with MSConvert, and routing the mzML into MS-DIAL for quantification and annotation of user-defined targets so Rapid QC-MS can report QC metrics.

## When to use

Use when a completed Thermo .raw file is written during an active instrument run and must be converted to mzML (MSConvert) then quantified/annotated (MS-DIAL) for automated QC.

## When NOT to use

- Processing raw files from non-Thermo vendors (Agilent, Bruker, Sciex, Waters) — Rapid QC-MS has only been tested extensively on Thermo acquisition sequences and Thermo RAW files, and bugs are expected with other vendor formats
- Running the conversion/processing stage on MacOS — MSConvert and MS-DIAL dependencies mean MacOS users can only monitor/view instrument run data, not process files
- Input data is already in mzML or a processed feature/quantification table — no raw vendor conversion stage is needed

## Inputs

- Completed Thermo .raw instrument files written during an active acquisition sequence
- Thermo acquisition sequence .csv (e.g. from the supplementary Zenodo deposit 10.5281/zenodo.10525183)
- Target spectral library .msp file

## Outputs

- mzML file in the job's output directory
- MS-DIAL target quantification/annotation results for user-defined targets
- QC metrics (instrument sensitivity, in-run retention time shifts, mass accuracy) surfaced on the Rapid QC-MS dashboard

## How to apply

Set up a Rapid QC-MS job to watch the acquisition directory for newly completed Thermo raw files from the instrument sequence (.csv), then invoke MSConvert (version 3.0.23332-7faa7f422) to convert each completed .raw into open mzML format. Route the converted mzML file into MS-DIAL (ver.4.9.22121823) to carry out quantification and annotation of all user-defined targets specified in the library (.msp). Write both the mzML output and the MS-DIAL target quantification/annotation results to the job's output directory. The rationale for the vendor-to-mzML conversion step is that MSConvert converts raw acquired data into the open mzML format before routing it to the data processing pipeline, making the workflow vendor-agnostic. Validate by confirming the mzML file exists in the expected format and that MS-DIAL produced quantified/annotated results for the defined targets, feeding the dashboard metrics (instrument sensitivity, in-run retention time shifts, mass accuracy drift) within minutes of file write.

## Related tools

- **Rapid QC-MS** (Orchestrates the job: watches for newly written raw files, invokes MSConvert, routes mzML to MS-DIAL, and writes results to the output directory) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert 3.0.23332-7faa7f422** (Converts each completed Thermo .raw file into open mzML format before routing it to the data processing pipeline) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **MS-DIAL ver.4.9.22121823** (Processes the converted mzML files, carrying out quantification and annotation for all user-defined targets) — http://prime.psc.riken.jp/compms/msdial/main.html

## Examples

```
rapidqcms
```

## Evaluation signals

- An mzML file exists in the job's output directory for each completed Thermo .raw file and is in the expected open mzML format
- MS-DIAL produced quantification/annotation results for all user-defined targets from the .msp library
- Dashboard QC results (instrument sensitivity, in-run retention time shifts, mass accuracy drift) update within minutes of the new data file being written
- Conversion is invoked only for completed raw files during the active job, not for files still being written
- Tool versions match the paper's reported stack: MSConvert 3.0.23332-7faa7f422 and MS-DIAL ver.4.9.22121823, both installed manually

## Limitations

- Rapid QC-MS was designed to run on Windows platforms because of its MSConvert and MS-DIAL dependencies; MacOS users can only monitor/view instrument run data
- Only tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files — other vendor formats may have bugs
- MSConvert and MS-DIAL must be downloaded and installed manually; only Python 3.8–3.11 dependencies (Pandas, SQLAlchemy, Plotly Dash, Watchdog, etc.) are installed automatically

## Evidence

- [readme] Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert for vendor format data conversion and MS-DIAL for data processing and identification.: [evidence span withheld — anchor status: not_found]
