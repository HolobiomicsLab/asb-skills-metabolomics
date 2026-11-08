---
name: raw-file-format-conversion-to-mzml
description: Use when processing newly written instrument raw files (e.g. Thermo RAW)
  that must be converted to mzML before feature extraction, QC metric computation,
  or dashboard reporting.
when_to_use_negative:
- You already hold an open-format mzML file and only need MS-DIAL processing or dashboard
  viewing — conversion is unnecessary and Rapid QC-MS on macOS can still monitor/view
  data without vendor conversion
- The goal is to monitor/view instrument run data without processing raw files — Rapid
  QC-MS on MacOS can monitor/view without MSConvert
- You are processing Agilent, Bruker, Sciex, or Waters data — support is undeveloped
  and untested, and the authors request collaboration rather than promising correct
  conversion
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MSConvert
  role: Converts raw acquired data into open mzML format before routing it to the
    Rapid QC-MS data-processing pipeline; version 3.0.23332-7faa7f422
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: Rapid QC-MS
  role: 'Orchestrates the conversion step: watches for newly written raw files, invokes
    MSConvert, and routes mzML output to MS-DIAL for quantification/annotation'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL
  role: Consumes the converted mzML files and performs quantification and annotation
    procedures for user-defined targets; ver.4.9.22121823
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:bdab44799482c922b9bf0e4a66a4a5ea99a451aaaa0c90e7aa674e420272774e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/czbiohub-sf/Rapid-QC-MS
  tools:
  - MS-DIAL
  - MSConvert
  - MSConvert 3.0.23332-7faa7f422
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# raw-file-format-conversion-to-mzml

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert vendor-format LC-MS/MS raw files (e.g. Thermo RAW) into open mzML format using MSConvert so downstream processing such as MS-DIAL quantification/annotation and Rapid QC-MS quality checks can proceed. This is the format-normalization step that lets Rapid QC-MS work across vendor formats.

## When to use

Use when processing newly written instrument raw files (e.g. Thermo RAW) that must be converted to mzML before feature extraction, QC metric computation, or dashboard reporting.

## When NOT to use

- You already hold an open-format mzML file and only need MS-DIAL processing or dashboard viewing — conversion is unnecessary and Rapid QC-MS on macOS can still monitor/view data without vendor conversion
- The goal is to monitor/view instrument run data without processing raw files — Rapid QC-MS on MacOS can monitor/view without MSConvert
- You are processing Agilent, Bruker, Sciex, or Waters data — support is undeveloped and untested, and the authors request collaboration rather than promising correct conversion

## Inputs

- Vendor-format LC-MS/MS raw file (e.g. Thermo .raw) written during an active Rapid QC-MS job
- MSConvert installation (version 3.0.23332-7faa7f422)

## Outputs

- Open-format mzML file routed to the downstream data-processing pipeline (MS-DIAL quantification/annotation)

## How to apply

As part of a Rapid QC-MS job, invoke MSConvert (version 3.0.23332-7faa7f422) on each completed raw file to convert vendor data into open mzML format; Rapid QC-MS then routes the converted mzML into the data-processing pipeline where MS-DIAL (ver.4.9.22121823) performs quantification and annotation of user-defined targets from the .msp library. The rationale is that MSConvert converts raw acquired data into open mzML before routing it to the pipeline, making Rapid QC-MS vendor-agnostic (only Thermo sequences/RAW have been extensively tested). MSConvert and MS-DIAL are not installed by the pip package and must be installed manually; Rapid QC-MS is Windows-oriented because of these dependencies. Validate by confirming the mzML file exists in the expected format and that MS-DIAL produced quantified/annotated results for the defined targets.

## Related tools

- **MSConvert** (Converts raw acquired data into open mzML format before routing it to the Rapid QC-MS data-processing pipeline; version 3.0.23332-7faa7f422) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **Rapid QC-MS** (Orchestrates the conversion step: watches for newly written raw files, invokes MSConvert, and routes mzML output to MS-DIAL for quantification/annotation) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL** (Consumes the converted mzML files and performs quantification and annotation procedures for user-defined targets; ver.4.9.22121823) — http://prime.psc.riken.jp/compms/msdial/main.html

## Examples

```
rapidqcms  # Rapid QC-MS watches for new .raw files, invokes MSConvert 3.0.23332-7faa7f422 to produce mzML, then runs MS-DIAL ver.4.9.22121823
```

## Evaluation signals

- Confirm the mzML file exists in the expected format in the job output directory
- Confirm MS-DIAL produced quantified/annotated results for the user-defined targets from the .msp library
- Verify MSConvert version matches 3.0.23332-7faa7f422 as documented for Rapid QC-MS integration
- Check that conversion occurred only for completed raw files, not partially written files

## Limitations

- MSConvert and MS-DIAL must be installed manually; the pip install of rapid-qc-ms only adds Python dependencies
- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files — other vendor formats may hit bugs
- Rapid QC-MS is designed to run on Windows because of its MSConvert dependency; MacOS users can only monitor/view data
- MSConvert must have already been run before data quality assessment because it requires processed data files rather than raw files

## Evidence
