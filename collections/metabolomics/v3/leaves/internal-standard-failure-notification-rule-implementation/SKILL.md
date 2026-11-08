---
name: internal-standard-failure-notification-rule-implementation
description: Use when implementing or configuring automated notification logic in
  Rapid QC-MS that must alert designated researchers whenever a newly acquired raw
  file's internal standard quality metrics violate user-defined IS Configurations
  criteria.
when_to_use_negative:
- The analysis goal is interactive inspection of IS retention time, m/z, or intensity
  drifts on the Rapid QC-MS dashboard rather than automated rule-based alerting.
- Notification logic is needed for RS or ES (reference standard / extraction standard)
  quality-control failures rather than internal standard failures defined in the IS
  Configurations settings.
- Data acquisition is not running on a Windows platform with MSConvert and MS-DIAL
  installed, since Rapid QC-MS's automated processing pipeline that generates the
  IS metrics depends on those manual dependencies.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3360
tools:
- name: Rapid QC-MS
  role: Host software that invokes the notification module after each specimen's QC
    evaluation and supplies the IS metrics and IS Configurations criteria
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Slack API
  role: Python dependency used as one supported communication mode to dispatch IS
    failure alerts to designated researchers
- name: Google API
  role: Python dependency used as the email-based supported communication mode for
    dispatching IS failure alerts
- name: MS-DIAL
  role: Manually installed dependency that processes converted data to identify internal
    standards, producing the IS metrics the failure rule evaluates
- name: MSConvert
  role: Manually installed dependency that converts vendor RAW data into open mzML
    format before the processing pipeline that generates IS metrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:1380a72ff2017e2e999d52401e3c772c6ea9549129e74eca6fbca6b60ce82c44
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# internal-standard-failure-notification-rule-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements Rapid QC-MS's semiautonomous alerting rule: when a specimen's internal standards fail the criteria defined in the IS Configurations settings, dispatch a failure notification to all designated researchers via a supported messaging integration such as Slack or email.

## When to use

Use when implementing or configuring automated notification logic in Rapid QC-MS that must alert designated researchers whenever a newly acquired raw file's internal standard quality metrics violate user-defined IS Configurations criteria.

## When NOT to use

- The analysis goal is interactive inspection of IS retention time, m/z, or intensity drifts on the Rapid QC-MS dashboard rather than automated rule-based alerting.
- Notification logic is needed for RS or ES (reference standard / extraction standard) quality-control failures rather than internal standard failures defined in the IS Configurations settings.
- Data acquisition is not running on a Windows platform with MSConvert and MS-DIAL installed, since Rapid QC-MS's automated processing pipeline that generates the IS metrics depends on those manual dependencies.

## Inputs

- IS Configurations settings file (user-defined internal standard acceptance criteria and designated researcher contact list)
- Internal standard quality metrics computed by Rapid QC-MS for the newly acquired specimen raw file (IS retention time, m/z, intensity)
- Specimen identifiers from the newly written RAW/mzML data file

## Outputs

- Failure notification message dispatched to all designated researchers via Slack API or email (Google API)
- Silent no-op (no notification) when IS criteria are met

## How to apply

Write a reusable module that Rapid QC-MS invokes after each specimen's QC evaluation. Load the IS Configurations settings to read the user-defined internal standard acceptance criteria and the list of designated researcher contacts. Compare the IS quality metrics computed for the newly acquired raw file (e.g. internal standard retention time, m/z, and intensity tracked across samples) against those criteria to decide whether the specimen fails. On failure, construct an alert containing the specimen identifiers and the offending IS metrics, then dispatch it to all designated researchers through the supported communication integrations installed as Python dependencies during setup — the Slack API or the Google (email) API. The rule remains silent when the criteria are met, so Rapid QC-MS 'can operate in a semiautonomous fashion, alerting users to specimens with potentially poor analytical integrity'.

## Related tools

- **Rapid QC-MS** (Host software that invokes the notification module after each specimen's QC evaluation and supplies the IS metrics and IS Configurations criteria) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Slack API** (Python dependency used as one supported communication mode to dispatch IS failure alerts to designated researchers)
- **Google API** (Python dependency used as the email-based supported communication mode for dispatching IS failure alerts)
- **MS-DIAL** (Manually installed dependency that processes converted data to identify internal standards, producing the IS metrics the failure rule evaluates)
- **MSConvert** (Manually installed dependency that converts vendor RAW data into open mzML format before the processing pipeline that generates IS metrics)

## Examples

```
py -m pip install rapid-qc-ms && rapidqcms
```

## Evaluation signals

- A specimen whose IS metrics violate the IS Configurations criteria triggers a notification to every designated researcher through the configured messaging integration.
- A specimen whose IS metrics meet the criteria produces no notification (rule is silent on pass).
- The notification message contains the specimen identifiers and the offending internal standard metrics.
- Alerts are delivered via a supported communication mode (Slack API or Google/email API), matching the paper's statement about 'frequently used messaging applications'.
- The alerting logic is written as a reusable module invoked by Rapid QC-MS after each new raw file's QC evaluation, consistent with reporting metrics 'once each new raw file has been collected'.

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may produce buggy IS metric results that corrupt failure decisions.
- Automated processing that generates the IS metrics requires MSConvert and MS-DIAL to be installed manually; only Windows platforms support this, so notification triggering is limited on MacOS (which can only monitor/view run data).
- Alerting is limited to the supported communication integrations shipped as Python dependencies (Slack API and Google API); other messaging channels are not supported out of the box.

## Evidence

- [other] When an internal standard for a specimen fails the IS Configurations criteria, Rapid QC-MS automatically sends a failure notification through a supported communication mode to all designated researchers.: [evidence span withheld — anchor status: not_found]
- [readme] MSConvert and MS-DIAL must be installed manually; Rapid QC-MS was designed for Windows because of these dependencies: [evidence span withheld — anchor status: not_found]
