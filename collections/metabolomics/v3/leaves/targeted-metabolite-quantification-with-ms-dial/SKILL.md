---
name: targeted-metabolite-quantification-with-ms-dial
description: Use when a completed vendor raw file (e.g., Thermo .raw) from an active
  LC-MS/MS run must be converted to mzML and its user-defined targets quantified/annotated
  as part of Rapid QC-MS quality control.
when_to_use_negative:
- Input is already an mzML or other open-format file that bypasses MSConvert conversion
  — this skill targets vendor raw files (only extensively tested on Thermo .raw).
- Goal is monitoring/viewing instrument run data on MacOS, where the MSConvert/MS-DIAL-dependent
  processing cannot run (Rapid QC-MS is designed for Windows).
- Only MSConvert conversion is needed without downstream targeted quantification/annotation
  — this skill covers the full raw-to-MS-DIAL stage.
- ''
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Rapid QC-MS
  role: 'Orchestrates the job: watches for newly written raw files and routes converted
    mzML into MS-DIAL for target quantification/annotation'
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert 3.0.23332-7faa7f422
  role: Converts each completed Thermo raw file into the open mzML format before routing
    to the data processing pipeline
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: MS-DIAL ver.4.9.22121823
  role: Processes the converted mzML files, carrying out quantification and annotation
    of all user-defined targets
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:d66b97cdf7544e8c79108892a9937da0be2173f02eeafa77399ead64f151c939
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
  - MS-DIAL ver.4.9.22121823
  - MSConvert
  - MSConvert 3.0.23332-7faa7f422
  - Rapid QC-MS
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Targeted metabolite quantification with MS-DIAL

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert completed Thermo .raw files to open mzML via MSConvert and process the mzML with MS-DIAL to perform quantification and annotation of user-defined targets, feeding QC metrics (instrument sensitivity, retention time shifts, mass accuracy) to the Rapid QC-MS dashboard.

## When to use

Use when a completed vendor raw file (e.g., Thermo .raw) from an active LC-MS/MS run must be converted to mzML and its user-defined targets quantified/annotated as part of Rapid QC-MS quality control.

## When NOT to use

- Input is already an mzML or other open-format file that bypasses MSConvert conversion — this skill targets vendor raw files (only extensively tested on Thermo .raw).
- Goal is monitoring/viewing instrument run data on MacOS, where the MSConvert/MS-DIAL-dependent processing cannot run (Rapid QC-MS is designed for Windows).
- Only MSConvert conversion is needed without downstream targeted quantification/annotation — this skill covers the full raw-to-MS-DIAL stage.
- 

## Inputs

- Completed Thermo .raw files written during an active acquisition sequence
- Sequence .csv defining the acquisition/job targets
- Spectral library .msp file of user-defined targets
- Example data from Metabolomics Workbench (DOI 10.21228/M8Z119) and Zenodo supplementary files (10.5281/zenodo.10525183)

## Outputs

- mzML-converted file for each raw file, written to the job's output directory
- MS-DIAL target quantification and annotation results written to the job's output directory
- QC metrics (instrument sensitivity, in-run retention time shifts, mass accuracy drift) reported on the Rapid QC-MS dashboard

## How to apply

Configure a Rapid QC-MS job to watch for newly written Thermo .raw files, then invoke MSConvert (version 3.0.23332-7faa7f422) to convert each completed raw file into the open mzML format. Route the converted mzML file into MS-DIAL (ver.4.9.22121823), which must be downloaded and installed separately, to carry out quantification and annotation for all user-defined targets. Provide the targets via a sequence .csv and a spectral library .msp from the supplementary Zenodo deposit (10.5281/zenodo.10525183) alongside example data from Metabolomics Workbench (DOI 10.21228/M8Z119). Write the mzML output and MS-DIAL target quantification/annotation results to the job's output directory, where the QC metrics are computed and surfaced on the browser-viewable dashboard within minutes. The rationale is that converting to vendor-neutral mzML lets MS-DIAL process any vendor format, making the QC pipeline universal while validated primarily on Thermo data.

## Related tools

- **Rapid QC-MS** (Orchestrates the job: watches for newly written raw files and routes converted mzML into MS-DIAL for target quantification/annotation) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert 3.0.23332-7faa7f422** (Converts each completed Thermo raw file into the open mzML format before routing to the data processing pipeline) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **MS-DIAL ver.4.9.22121823** (Processes the converted mzML files, carrying out quantification and annotation of all user-defined targets) — http://prime.psc.riken.jp/compms/msdial/main.html

## Examples

```
rapidqcms
```

## Evaluation signals

- Confirm the mzML file exists in the expected format in the job's output directory after conversion.
- Confirm MS-DIAL produced quantified/annotated results for the defined targets.
- Verify results are consistent with the QC metrics Rapid QC-MS reports (instrument sensitivity, retention time shifts, mass accuracy).
- Confirm the dashboard was updated with QC results within minutes of the new data file being written.
- Verify QC-fail notifications (Slack or email) fire when a newly acquired raw file does not meet specified requirements.

## Limitations

- MS-DIAL (ver.4.9.22121823) and MSConvert must be downloaded and installed manually; they are not installed by the Rapid QC-MS pip setup.
- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs.
- Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert and MS-DIAL; MacOS users can only monitor/view run data.
- The MSConvert version tested is 3.0.23332-7faa7f422; other versions are unverified.

## Evidence

- [methods] Completed raw files are automatically converted to open mzML format via MSConvert, and MS-DIAL then processes the mzML files: [evidence span withheld — anchor status: not_found]
