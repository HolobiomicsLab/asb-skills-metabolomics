---
name: internal-standard-qc-flagging
description: Use when implementing or re-implementing per-internal-standard QC status
  assignment for LC-MS/MS specimens, where each IS target needs Pass/Warn/Fail flags
  from RT and m/z deviation thresholds and missing-peak detection.
when_to_use_negative:
- Input is already an aggregated feature table or processed QC report — this skill
  operates on per-IS peak measurements from the mzML/MS-DIAL identification stage,
  not on summary tables.
- The goal is instrument-level or analysis-level QC (sensitivity, drift curves) rather
  than per-internal-standard Pass/Warn/Fail status assignment within a single specimen.
- Data cannot be converted by MSConvert to mzML (e.g. acquisition sequences from Agilent,
  Bruker, Sciex, or Waters are not yet supported).
tools:
- name: Rapid QC-MS
  role: Host package whose rule module implements the IS Pass/Warn/Fail decision function
    and surfaces results on the QC dashboard
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: MSConvert
  role: Converts vendor raw data into open mzML format before routing to the data
    processing pipeline that feeds the IS QC rules
- name: MS-DIAL
  role: Data processing and identification dependency that produces the per-IS peak
    detections used by the status rules
- name: Python
  role: Language used to encode the pure decision function (versions 3.8–3.11) and
    pytest boundary tests
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:d5db70d7e15e259e23cfd6ae1556d5b92a6817ad274b0aa4b9b7cb0232604e72
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# internal-standard-qc-flagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes Rapid QC-MS's per-internal-standard status rules: each IS target in a specimen is assigned 'Pass', 'Warn', or 'Fail' based on retention-time shift from library (Warn if ΔRT exceeds 66% of the RT shift-from-library threshold), in-run RT shift (Warn if >80% of the in-run average threshold), m/z shift from library (Warn if Δm/z exceeds 80% of the threshold), and missing peaks (Fail).

## When to use

Use when implementing or re-implementing per-internal-standard QC status assignment for LC-MS/MS specimens, where each IS target needs Pass/Warn/Fail flags from RT and m/z deviation thresholds and missing-peak detection.

## When NOT to use

- Input is already an aggregated feature table or processed QC report — this skill operates on per-IS peak measurements from the mzML/MS-DIAL identification stage, not on summary tables.
- The goal is instrument-level or analysis-level QC (sensitivity, drift curves) rather than per-internal-standard Pass/Warn/Fail status assignment within a single specimen.
- Data cannot be converted by MSConvert to mzML (e.g. acquisition sequences from Agilent, Bruker, Sciex, or Waters are not yet supported).

## Inputs

- Per-IS-target measured retention time and library RT (to compute RT shift-from-library)
- In-run measured RT and in-run average RT (to compute in-run ΔRT)
- Per-IS-target measured m/z and library m/z (to compute Δm/z)
- Library RT-shift threshold, in-run average RT threshold, and m/z shift threshold from the user-defined QC job configuration
- Peak-detection result per IS target (detected / missing), derived from mzML-converted data processed by MS-DIAL

## Outputs

- Per-internal-standard status string: 'Pass', 'Warn' (Warning), or 'Fail'
- Per-specimen IS QC table feeding the browser-viewable dashboard and Slack/email notifications

## How to apply

Read the paper's threshold fractions from the methods text (66% of the RT shift-from-library threshold, 80% of the in-run average RT threshold, 80% of the m/z shift-from-library threshold) and encode them as a pure Python decision function in the rapidqcms rule module, taking measured RT, in-run RT, m/z, and the corresponding library/in-run thresholds as arguments. The decision logic is ordered: an IS target with no detected peak is always 'Fail'; otherwise any single threshold breach at the stated fraction (0.66 for library RT shift, 0.80 for in-run RT shift, 0.80 for m/z shift) raises 'Warning', and an IS meeting all three is 'Pass'. In the broader pipeline, IS intensities, retention times, and m/z values are measured from vendor raw data that MSConvert first converts to open mzML format before MS-DIAL-based identification feeds the QC rules. Write pytest unit tests covering each boundary (at, just-below, and just-above each 66%/80% fraction) plus the missing-IS fail case, and run pytest to confirm all rules behave as specified.

## Related tools

- **Rapid QC-MS** (Host package whose rule module implements the IS Pass/Warn/Fail decision function and surfaces results on the QC dashboard) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **MSConvert** (Converts vendor raw data into open mzML format before routing to the data processing pipeline that feeds the IS QC rules)
- **MS-DIAL** (Data processing and identification dependency that produces the per-IS peak detections used by the status rules)
- **Python** (Language used to encode the pure decision function (versions 3.8–3.11) and pytest boundary tests)

## Examples

```
status = "Fail" if not detected else ("Warn" if d_rt > 0.66*rt_thr or d_rt_run > 0.80*run_thr or d_mz > 0.80*mz_thr else "Pass")
```

## Evaluation signals

- Every boundary test case matches the paper's stated fractions exactly: 'Warning' triggers above 0.66× the library RT-shift threshold and above 0.80× the in-run RT and m/z thresholds, with no tolerance drift.
- The missing-IS case always returns 'Fail' regardless of RT or m/z values.
- An IS within all three thresholds returns 'Pass'; a single threshold breach returns 'Warning' (rules are independent, not cumulative).
- All pytest tests (at, just-below, and just-above each boundary plus missing-IS) pass.
- Result statuses propagate to the dashboard/notifications consistent with Rapid QC-MS's reported per-specimen fitness output.

## Limitations

- Rapid QC-MS was designed for Windows due to its MSConvert and MS-DIAL dependencies; MacOS users can only monitor/view instrument run data, not run the full pipeline.
- Extensively tested only on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may contain bugs.
- Warn thresholds are fractional (66%/80%) of user-configurable library and in-run thresholds, so absolute cutoffs depend on the QC job configuration and must be read from the methods/configuration, not hardcoded globally.

## Evidence

- [discussion] By automatically evaluating numerous parameters within minutes of writing a specimen's raw file, Rapid QC-MS allows users to quickly and easily determine the fitness of the LC-MS/MS, each specimen,: [evidence span withheld — anchor status: not_found]
- [readme] Rapid QC-MS was designed to run on Windows platforms because of its dependency on MSConvert for vendor format data conversion and MS-DIAL for data processing and identification.: [evidence span withheld — anchor status: not_found]
