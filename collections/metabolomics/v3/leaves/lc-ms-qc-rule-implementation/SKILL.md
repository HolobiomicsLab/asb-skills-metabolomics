---
name: lc-ms-qc-rule-implementation
description: Use when a dashboard or QC pipeline needs to escalate per-internal-standard
  QC results into a specimen-level 'Warning' label using Rapid QC-MS's dropout-threshold
  and IS-Warn-fraction rule.
when_to_use_negative:
- Input is already a specimen-level label table with no per-internal-standard intensity
  or designation data to aggregate
- Data was not processed through the Rapid QC-MS pipeline (MSConvert raw-to-mzML conversion
  plus MS-DIAL processing), so IS dropout counts and per-IS QC designations are unavailable
- The goal is per-raw-file alerting or notification (Slack/email) rather than reproducing
  the specimen-level escalation logic itself
tools:
- name: Rapid QC-MS
  role: Source package whose specimen-level warning evaluation routine is reimplemented
    and whose Specimen Table displays the resulting 'Warning' labels
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Python
  role: Language used to implement the two warning conditions and their OR combination;
    Rapid QC-MS supports Python 3.8–3.11
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:bde357a5e9f280324e43cf74b675097c53d73821af6272627e71e4a9ca57fe43
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
  - Python
  - Rapid QC-MS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lc-ms-qc-rule-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reimplements Rapid QC-MS's specimen-level 'Warn' escalation rule, in which a specimen is flagged 'Warning' in the dashboard Specimen Table when either IS intensity dropouts reach 75% of the user-defined threshold or >=50% of its internal standards individually carry a 'Warn' designation.

## When to use

Use when a dashboard or QC pipeline needs to escalate per-internal-standard QC results into a specimen-level 'Warning' label using Rapid QC-MS's dropout-threshold and IS-Warn-fraction rule.

## When NOT to use

- Input is already a specimen-level label table with no per-internal-standard intensity or designation data to aggregate
- Data was not processed through the Rapid QC-MS pipeline (MSConvert raw-to-mzML conversion plus MS-DIAL processing), so IS dropout counts and per-IS QC designations are unavailable
- The goal is per-raw-file alerting or notification (Slack/email) rather than reproducing the specimen-level escalation logic itself

## Inputs

- Rapid QC-MS source code (github.com/czbiohub-sf/Rapid-QC-MS clone or Zenodo 10.5281/zenodo.10525183 archive)
- Per-specimen internal standard intensity dropout counts
- User-defined IS dropout threshold (configurable parameter)
- Per-internal-standard QC designations ('Pass'/'Warn') for each specimen's IS targets

## Outputs

- Specimen-level 'Warning' (or absence thereof) flags rendered in the dashboard Specimen Table
- Validated rule module with boundary-crossing test cases

## How to apply

Clone the Rapid QC-MS source from github.com/czbiohub-sf/Rapid-QC-MS and locate the specimen-level warning evaluation routine in the package's quality-control logic (the package is written in Python, compatible with versions 3.8–3.11). Implement condition 1: flag 'Warning' when the count of IS intensity dropouts is >= 75% of the user-defined threshold, keeping that threshold configurable. Implement condition 2: flag 'Warning' when >= 50% of the specimen's IS targets individually carry a 'Warn' QC designation. Combine both conditions with OR logic so either trigger alone produces a specimen 'Warning' in the Specimen Table; the rationale is that Rapid QC-MS evaluates numerous parameters per specimen within minutes of writing each raw file, so per-IS signals must aggregate into an interpretable specimen-level label.

## Related tools

- **Rapid QC-MS** (Source package whose specimen-level warning evaluation routine is reimplemented and whose Specimen Table displays the resulting 'Warning' labels) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Python** (Language used to implement the two warning conditions and their OR combination; Rapid QC-MS supports Python 3.8–3.11)

## Examples

```
git clone https://github.com/czbiohub-sf/Rapid-QC-MS && py -3.10 -m pip install rapid-qc-ms
```

## Evaluation signals

- A test specimen whose IS dropout count is exactly 75% of the user-defined threshold is flagged 'Warning'
- A test specimen with >= 50% of its internal standards carrying a 'Warn' designation is flagged 'Warning' even if the dropout condition is not met (OR-logic check)
- Specimens below both boundaries (e.g., dropouts well under 75% of threshold and < 50% of IS in 'Warn') produce no 'Warning' label
- The IS dropout threshold remains user-configurable and changes in its value move the boundary accordingly
- The rule module executes without error within the Rapid QC-MS QC logic under Python 3.8–3.11

## Limitations

- Rapid QC-MS was designed for Windows platforms due to dependencies on MSConvert and MS-DIAL; MacOS users can monitor/view data but the full processing pipeline may not run
- The tool has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may contain bugs
- Dependencies such as MSConvert and MS-DIAL must be installed manually and are not installed by the Python package setup

## Evidence

- [results] A specimen receives a specimen-level 'Warning' label when either the number of IS intensity dropouts is 75% or more of the user-defined threshold: [evidence span withheld — anchor status: not_found]
