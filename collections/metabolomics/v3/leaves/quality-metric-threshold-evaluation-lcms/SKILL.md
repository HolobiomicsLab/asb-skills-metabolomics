---
name: quality-metric-threshold-evaluation-lcms
description: Use when a newly written Thermo RAW file must be judged against IS Configurations
  thresholds so that specimens failing internal standard criteria are automatically
  flagged to researchers during an acquisition.
when_to_use_negative:
- Input is already a finished QC report or feature table rather than a newly acquired
  raw file awaiting threshold evaluation
- The goal is retrospective batch QC visualization (e.g. building drift plots over
  months) rather than synchronous per-file fail notification during an instrument
  run
- Non-Thermo vendor data where Rapid QC-MS processing has not been validated and may
  contain bugs
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Rapid QC-MS
  role: Hosts the QC evaluation loop that applies IS Configurations thresholds and
    invokes the notification module after each specimen's QC evaluation
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Slack API
  role: Dispatches the specimen failure notification to designated researchers through
    a supported messaging integration
- name: Google API
  role: Supports email notification dispatch and Google Drive remote QC sync as an
    alternative communication mode
- name: MSConvert
  role: Converts the newly acquired vendor RAW data into open mzML format before routing
    to the data processing pipeline used for IS metric computation
- name: MS-DIAL
  role: Data processing and identification backend producing the IS metrics that are
    evaluated against IS Configurations thresholds
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:651ddf25effedb2596941287e3eef21c065c2b476601ee91d2f2f88d62509488
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
  - Google API
  - MS-DIAL
  - MSConvert
  - Rapid QC-MS
  - Slack API
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# quality-metric-threshold-evaluation-lcms

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate LC-MS/MS quality metrics against user-defined acceptance criteria (IS Configurations) and trigger automatic notifications to designated researchers whenever a newly acquired raw file fails, using Rapid QC-MS's Slack or email alerting path.

## When to use

Use when a newly written Thermo RAW file must be judged against IS Configurations thresholds so that specimens failing internal standard criteria are automatically flagged to researchers during an acquisition.

## When NOT to use

- Input is already a finished QC report or feature table rather than a newly acquired raw file awaiting threshold evaluation
- The goal is retrospective batch QC visualization (e.g. building drift plots over months) rather than synchronous per-file fail notification during an instrument run
- Non-Thermo vendor data where Rapid QC-MS processing has not been validated and may contain bugs

## Inputs

- IS Configurations file containing user-defined acceptance criteria and designated researcher contacts
- Newly acquired Thermo RAW file converted to open mzML format
- IS quality metrics computed per specimen (retention time, m/z, intensity)

## Outputs

- Pass/fail decision for each specimen against IS Configurations criteria
- Failure notification message (Slack or email) to all designated researchers, containing specimen identifiers and offending IS metrics

## How to apply

Load the IS Configurations file to read the user-defined acceptance criteria and the list of designated researcher contacts. Evaluate the IS quality metrics computed for the newly acquired raw file (converted to mzML by MSConvert and processed by MS-DIAL) against the IS Configurations criteria to decide whether the specimen fails. On failure, construct an alert message containing the specimen identifiers and the offending IS metrics, then dispatch it to all designated researchers through the supported communication integrations (Slack API or email via Google API) installed during Rapid QC-MS setup. Keep this logic as a reusable module invoked after each specimen's QC evaluation, consistent with the paper's statement that the tool 'alerts users whenever a newly acquired raw file does not meet specified requirements'. Silent pass (no notification) is the correct behavior when criteria are met.

## Related tools

- **Rapid QC-MS** (Hosts the QC evaluation loop that applies IS Configurations thresholds and invokes the notification module after each specimen's QC evaluation) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Slack API** (Dispatches the specimen failure notification to designated researchers through a supported messaging integration)
- **Google API** (Supports email notification dispatch and Google Drive remote QC sync as an alternative communication mode)
- **MSConvert** (Converts the newly acquired vendor RAW data into open mzML format before routing to the data processing pipeline used for IS metric computation)
- **MS-DIAL** (Data processing and identification backend producing the IS metrics that are evaluated against IS Configurations thresholds)

## Examples

```
rapidqcms
```

## Evaluation signals

- The module programmatically triggers a notification for a specimen whose IS metrics violate the configured criteria and remains silent when criteria are met
- Notification is delivered to all designated researchers listed in the IS Configurations file, not just a single recipient
- Alert message contains both specimen identifiers and the specific IS metrics that breached thresholds
- No notification is emitted for a newly acquired raw file that satisfies all IS Configurations criteria
- Dispatch path uses the supported communication integrations (Slack API or email/Google API) installed during Rapid QC-MS setup

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs
- Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert and MS-DIAL
- Rapid QC-MS is semiautonomous, not fully autonomous - alerts depend on predefined criteria and cannot replace manual judgment on ambiguous failures
- MSConvert and MS-DIAL must be installed manually; Python dependencies (including Google API and Slack API) are installed automatically but require Python 3.8-3.11

## Evidence

- [discussion] Automatically evaluate numerous parameters within minutes of writing a specimen's raw file: [evidence span withheld — anchor status: not_found]
