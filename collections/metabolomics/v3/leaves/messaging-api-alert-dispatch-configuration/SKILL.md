---
name: messaging-api-alert-dispatch-configuration
description: Use when setting up semi-autonomous QC during Thermo LC-MS/MS acquisition,
  so researchers are alerted minutes after a raw file fails internal standard acceptance
  criteria in IS Configurations.
when_to_use_negative:
- The user only needs retrospective QC review after data acquisition completes, rather
  than synchronous alerts during instrument runs
- Slack API and Google API dependencies were not installed during setup, or the deployment
  platform lacks the messaging integrations Rapid QC-MS expects
- The instrument is not a Thermo mass spectrometer — Rapid QC-MS has only been tested
  extensively on Thermo acquisition sequences and RAW files
tools:
- name: Rapid QC-MS
  role: Main package that watches newly acquired RAW files, evaluates IS metrics against
    IS Configurations, and dispatches alerts via messaging APIs
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Slack API
  role: Python dependency used as the supported communication mode for automatic QC-failure
    notifications to researchers
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Google API
  role: Python dependency enabling email-based alerting and Google Drive cloud sync
    of QC results
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Manually installed dependency converting raw acquired data into open mzML
    format before QC evaluation and alert logic
- name: MS-DIAL
  role: Manually installed dependency performing data processing and identification
    feeding the IS metrics assessed for alerting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:048ebd8161100d603baaaa337f453771025edc0825ea293e7f14ffd2b06cbbd3
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# messaging-api-alert-dispatch-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure Rapid QC-MS to dispatch automatic Slack or email notifications to designated researchers whenever a specimen's internal standard metrics fail the user-defined IS Configurations criteria during an LC-MS/MS run.

## When to use

Use when setting up semi-autonomous QC during Thermo LC-MS/MS acquisition, so researchers are alerted minutes after a raw file fails internal standard acceptance criteria in IS Configurations.

## When NOT to use

- The user only needs retrospective QC review after data acquisition completes, rather than synchronous alerts during instrument runs
- Slack API and Google API dependencies were not installed during setup, or the deployment platform lacks the messaging integrations Rapid QC-MS expects
- The instrument is not a Thermo mass spectrometer — Rapid QC-MS has only been tested extensively on Thermo acquisition sequences and RAW files

## Inputs

- User-defined IS Configurations file specifying internal standard acceptance criteria and designated researcher contacts
- Newly acquired Thermo RAW files monitored during instrument runs
- Rapid QC-MS job configuration (sequence .csv, library .msp from supplementary example files on Zenodo)

## Outputs

- Slack or email alert notifications sent to designated researchers on IS QC failure
- Browser-viewable dashboard update minutes after the new data file is written

## How to apply

Start Rapid QC-MS from the command line with `rapidqcms`, which runs the Python (3.8–3.11) package installed via `py -m pip install rapid-qc-ms` along with its Slack API and Google API dependencies. Load the IS Configurations file, which defines user-defined acceptance criteria for internal standard retention time, m/z, and intensity plus the list of designated researcher contacts. After each new Thermo RAW file is written, Rapid QC-MS converts it to mzML with MSConvert, processes it with MS-DIAL, and evaluates the IS metrics against the configured criteria. If a specimen fails, construct an alert message and dispatch it automatically through the supported communication mode — Slack or email — to all designated researchers, complementing the browser-viewable dashboard. Validate by confirming notifications fire only for specimens whose IS metrics violate the criteria and are silent otherwise.

## Related tools

- **Rapid QC-MS** (Main package that watches newly acquired RAW files, evaluates IS metrics against IS Configurations, and dispatches alerts via messaging APIs) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Slack API** (Python dependency used as the supported communication mode for automatic QC-failure notifications to researchers) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Google API** (Python dependency enabling email-based alerting and Google Drive cloud sync of QC results) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Manually installed dependency converting raw acquired data into open mzML format before QC evaluation and alert logic)
- **MS-DIAL** (Manually installed dependency performing data processing and identification feeding the IS metrics assessed for alerting)

## Examples

```
py -m pip install rapid-qc-ms; rapidqcms
```

## Evaluation signals

- A notification is programmatically triggered for a specimen whose IS metrics violate the IS Configurations criteria and remains silent when criteria are met
- Alerts reach all designated researchers listed in the IS Configurations through the configured communication mode (Slack or email)
- Alerts arrive minutes after a new data file is written, consistent with the synchronous QC claim
- The Slack/email dependencies (Slack API, Google API) are present in the Python environment installed during setup
- Dashboard and notifications stay consistent — no alert is issued for specimens that pass QC

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may have bugs
- Automatic alerting requires Windows platforms for MSConvert and MS-DIAL; MacOS users can only monitor/view run data
- MSConvert and MS-DIAL must be installed manually — they are not installed automatically with the pip package
- Rapid QC-MS runs in a semiautonomous fashion: alerts depend on user-defined criteria being correctly configured in IS Configurations

## Evidence
