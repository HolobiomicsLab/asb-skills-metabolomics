---
name: internal-standard-dropout-threshold-logic
description: Use when a specimen-level 'Warning' flag must be assigned in Rapid QC-MS
  based on IS intensity dropouts relative to a user-defined threshold or on per-IS
  QC designations.
when_to_use_negative:
- The input is not LC-MS/MS specimen data with internal standards (e.g., already-aggregated
  QC reports or feature tables) — this skill operates on per-specimen IS dropout counts
  and per-IS designations.
- The vendor format is non-Thermo and untested — the README states Rapid QC-MS 'has
  only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition
  sequences, and Thermo RAW files.'
- You only need per-file (run-level) QC rather than specimen-level escalation across
  a sequence.
edam_topics:
- http://edamontology.org/topic_3572
tools:
- name: Rapid QC-MS
  role: Host package whose quality-control logic contains the specimen-level warning
    evaluation routine and dashboard Specimen Table
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Python
  role: Implementation language for the warning-rule module (versions 3.8–3.11 supported)
- name: MSConvert
  role: Converts vendor raw data into open mzML format before routing to the data
    processing pipeline that yields IS intensities
  repo: https://proteowizard.sourceforge.io/tools/msconvert.html
- name: MS-DIAL
  role: Downstream data processing and identification that produces the internal standard
    intensity values evaluated by the escalation rule
  repo: http://prime.psc.riken.jp/compms/msdial/main.html
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:8ee7a1fb7bdca090eca7383e8475b0421a8443796f594e7f1c24c30c03dd649d
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
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# internal-standard-dropout-threshold-logic

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rapid QC-MS specimen-level escalation logic that assigns a 'Warning' label in the dashboard Specimen Table when internal standard (IS) intensity dropouts reach 75% of the user-defined threshold OR when 50% or more of a specimen's IS targets individually carry a 'Warn' designation.

## When to use

Use when a specimen-level 'Warning' flag must be assigned in Rapid QC-MS based on IS intensity dropouts relative to a user-defined threshold or on per-IS QC designations.

## When NOT to use

- The input is not LC-MS/MS specimen data with internal standards (e.g., already-aggregated QC reports or feature tables) — this skill operates on per-specimen IS dropout counts and per-IS designations.
- The vendor format is non-Thermo and untested — the README states Rapid QC-MS 'has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files.'
- You only need per-file (run-level) QC rather than specimen-level escalation across a sequence.

## Inputs

- Per-specimen LC-MS/MS raw file routed through MSConvert to mzML and processed via MS-DIAL
- Internal standard (IS) intensity values per specimen, from which dropouts are counted
- User-defined IS intensity dropout threshold (user-configurable)
- Per-IS individual QC designations ('Warn'/'Fail') for each IS target

## Outputs

- Specimen-level 'Warning' label in the Rapid QC-MS dashboard Specimen Table
- Optional Slack/email notifications alerting users to specimens with potentially poor analytical integrity

## How to apply

Load the Rapid QC-MS source from github.com/czbiohub-sf/Rapid-QC-MS and locate the specimen-level warning evaluation routine in the package's quality-control logic. Implement condition 1: flag 'Warning' when the number of IS intensity dropouts is >= 75% of the user-defined threshold. Implement condition 2: flag 'Warning' when >= 50% of the specimen's IS targets individually carry a 'Warn' designation. Combine the two conditions with OR logic so either trigger alone produces the specimen-level flag, keeping the IS dropout threshold user-configurable. The escalation complements Rapid QC-MS's per-file alerting of raw files that do not meet specified requirements, feeding the dashboard Specimen Table where specimens with potentially poor analytical integrity are surfaced.

## Related tools

- **Rapid QC-MS** (Host package whose quality-control logic contains the specimen-level warning evaluation routine and dashboard Specimen Table) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Python** (Implementation language for the warning-rule module (versions 3.8–3.11 supported))
- **MSConvert** (Converts vendor raw data into open mzML format before routing to the data processing pipeline that yields IS intensities) — https://proteowizard.sourceforge.io/tools/msconvert.html
- **MS-DIAL** (Downstream data processing and identification that produces the internal standard intensity values evaluated by the escalation rule) — http://prime.psc.riken.jp/compms/msdial/main.html

## Examples

```
rapidqcms
```

## Evaluation signals

- Test inputs with IS intensity dropouts >= 75% of the user-defined threshold produce a specimen 'Warning' designation.
- Test inputs with < 50% of IS targets carrying 'Warn' and dropouts below the 75%-of-threshold boundary produce no 'Warning' flag.
- Test inputs with >= 50% of IS targets individually designated 'Warn' produce a 'Warning' flag even when dropouts are below the dropout-threshold boundary (OR logic).
- The module executes without error on Rapid QC-MS's example data (Metabolomics Workbench DOI 10.21228/M8Z119 plus zenodo 10.5281/zenodo.10525183 supplementary sequence .csv, library .msp, and .raw files).
- The IS dropout threshold remains user-configurable — changing it changes the warning boundary accordingly.

## Limitations

- Rapid QC-MS was designed for Windows platforms due to MSConvert and MS-DIAL dependencies; MacOS users can monitor/view instrument run data but the full QC pipeline may not run.
- Only extensively tested on Thermo Fisher instruments, acquisition sequences, and RAW files; other vendor formats may contain bugs.
- The warning conditions are OR-combined, so either condition alone triggers escalation — borderline specimens near either boundary will be flagged.

## Evidence

- [abstract] Abstract: "Minutes after a new data file is written, a browser-viewable dashboard is updated with quality control results"
