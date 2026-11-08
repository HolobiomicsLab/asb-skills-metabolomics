---
name: specimen-level-qc-warning-assignment
description: Use when implementing or reproducing Rapid QC-MS's specimen-level QC
  logic — specifically assigning a dashboard Specimen Table 'Warning' label from IS
  intensity dropouts and per-IS 'Warn' flags rather than raw-file-level or instrument-level
  alerts.
when_to_use_negative:
- The goal is raw-file-level or instrument-level QC alerts (e.g. instrument sensitivity,
  retention time shift, mass accuracy drift for a single newly written .raw file)
  rather than per-specimen aggregation
- The data comes from non-Thermo vendors — Rapid QC-MS has only been tested extensively
  on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW
  files
- Input is already a processed feature table with specimen-level labels assigned,
  so no QC evaluation logic needs implementing
edam_topics:
- http://edamontology.org/topic_0601
tools:
- name: Rapid QC-MS
  role: Host package containing the specimen-level warning evaluation routine and
    the dashboard Specimen Table that displays the 'Warning' designation
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Python
  role: Implementation language (compatible with versions 3.8–3.11) for the warning-condition
    logic and boundary validation tests
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MS-DIAL
  role: Upstream data processing and identification step that produces the IS intensity
    values and per-IS QC results feeding the specimen-level conditions
- name: MSConvert
  role: Converts raw vendor data into open mzML format before routing it to the data
    processing pipeline used for QC evaluation
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:375351c28e032656aa0d86cc58e4ef77b48fa4f138d73bb4285bc6fce52c636d
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
  - Python
  - Rapid QC-MS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0601
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# specimen-level-qc-warning-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements Rapid QC-MS's specimen-level 'Warning' designation, which flags a specimen in the dashboard Specimen Table when internal standard (IS) intensity dropouts reach >= 75% of the user-defined threshold OR when >= 50% of the specimen's IS targets individually carry a QC 'Warn' designation.

## When to use

Use when implementing or reproducing Rapid QC-MS's specimen-level QC logic — specifically assigning a dashboard Specimen Table 'Warning' label from IS intensity dropouts and per-IS 'Warn' flags rather than raw-file-level or instrument-level alerts.

## When NOT to use

- The goal is raw-file-level or instrument-level QC alerts (e.g. instrument sensitivity, retention time shift, mass accuracy drift for a single newly written .raw file) rather than per-specimen aggregation
- The data comes from non-Thermo vendors — Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files
- Input is already a processed feature table with specimen-level labels assigned, so no QC evaluation logic needs implementing

## Inputs

- Per-specimen internal standard (IS) intensity measurements from MS-DIAL-processed mzML data
- Per-IS (individual) QC designations ('Warn'/'Pass') for each IS target
- User-defined IS intensity dropout threshold from the Rapid QC-MS QC job configuration (sequence .csv / job settings)

## Outputs

- Specimen-level 'Warning' designation rendered in the Rapid QC-MS dashboard Specimen Table
- Boundary-test results confirming warning flags at the 75%-of-threshold dropout and 50%-of-IS-Warn boundaries

## How to apply

Locate the specimen-level warning evaluation routine in the Rapid QC-MS Python package (github.com/czbiohub-sf/Rapid-QC-MS), which computes specimen QC after each specimen's raw file is processed through MSConvert (to mzML) and MS-DIAL. Implement two conditions against the user-defined IS dropout threshold: (1) flag 'Warning' when the number of IS intensity dropouts is >= 75% of that threshold, and (2) flag 'Warning' when >= 50% of the specimen's IS targets individually carry a 'Warn' QC designation. Combine both with OR logic so either trigger alone yields the specimen-level 'Warning'; keep the dropout threshold user-configurable so the 75% boundary scales with each job's settings. Validate by running the module on test inputs that cross each boundary (>=75% of threshold dropouts; >=50% IS-Warn) and confirming 'Warning' is produced while inputs below both boundaries remain unlabeled.

## Related tools

- **Rapid QC-MS** (Host package containing the specimen-level warning evaluation routine and the dashboard Specimen Table that displays the 'Warning' designation) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Python** (Implementation language (compatible with versions 3.8–3.11) for the warning-condition logic and boundary validation tests) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MS-DIAL** (Upstream data processing and identification step that produces the IS intensity values and per-IS QC results feeding the specimen-level conditions)
- **MSConvert** (Converts raw vendor data into open mzML format before routing it to the data processing pipeline used for QC evaluation) — https://proteowizard.sourceforge.io/tools/msconvert.html

## Examples

```
py -m pip install rapid-qc-ms && rapidqcms
```

## Evaluation signals

- A test specimen with IS intensity dropouts exactly at >= 75% of the user-defined threshold is labeled 'Warning', while one below 75% is not (when the second condition is not met)
- A specimen with >= 50% of its IS targets individually carrying 'Warn' is labeled 'Warning', while one below 50% is not (when the first condition is not met)
- Either condition alone triggers the flag — OR logic holds: a specimen below both boundaries must remain unlabeled
- The IS dropout threshold is read from user configuration, so changing it shifts the 75% boundary accordingly
- The module executes without error on test inputs and renders the 'Warning' designation in the dashboard Specimen Table

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may contain bugs
- The 75%-of-threshold dropout boundary depends on a user-defined threshold, so misconfiguration directly changes warning behavior
- MSConvert and MS-DIAL must be installed manually; they are not installed automatically with the Python package

## Evidence

- [discussion] By automatically evaluating numerous parameters within minutes of writing a specimen's raw file, Rapid QC-MS allows users to quickly and easily determine the fitness of the LC-MS/MS, each specimen, and the analysis: [evidence span withheld — anchor status: not_found]
