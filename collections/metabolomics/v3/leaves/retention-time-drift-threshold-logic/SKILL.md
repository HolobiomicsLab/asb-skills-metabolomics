---
name: retention-time-drift-threshold-logic
description: Use when coding or auditing per-target QC decision rules for internal
  standards in untargeted LC-MS/MS runs processed by Rapid QC-MS, and the paper's
  66%/80% fractional warning thresholds must be reproduced exactly.
when_to_use_negative:
- The task requires specimen- or analysis-level aggregate QC status rather than per-individual-internal-standard
  target status
- The lab uses warning thresholds other than the paper's 66%/80% fractions, e.g. locally
  recalibrated shift tolerances — the fractions must be re-read from the governing
  method, not copied
- Input is vendor raw data that has not yet been converted to mzML via MSConvert,
  so RT and m/z values per IS target are not yet available
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: Rapid QC-MS
  role: Hosts the rule module in which the pure IS-status decision function is implemented;
    the skill reproduces its per-target 'Fail'/'Warn'/'Pass' logic
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Python
  role: Language for the pure decision function and pytest boundary tests; Rapid QC-MS
    requires Python 3.8–3.11
- name: MS-DIAL
  role: Supplies peak detection and identification (RT, m/z) for internal-standard
    targets upstream of the threshold logic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:7ade3cde451bd2420ee11bd6d88564daf447633a370610852797f2bd2a610b38
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
  - Python
  - Rapid QC-MS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# retention-time-drift-threshold-logic

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements Rapid QC-MS's per-internal-standard status decision logic, assigning 'Fail', 'Warn', or 'Pass' based on ΔRT relative to the RT shift-from-library threshold, in-run ΔRT relative to the in-run average threshold, and Δm/z relative to the m/z shift-from-library threshold.

## When to use

Use when coding or auditing per-target QC decision rules for internal standards in untargeted LC-MS/MS runs processed by Rapid QC-MS, and the paper's 66%/80% fractional warning thresholds must be reproduced exactly.

## When NOT to use

- The task requires specimen- or analysis-level aggregate QC status rather than per-individual-internal-standard target status
- The lab uses warning thresholds other than the paper's 66%/80% fractions, e.g. locally recalibrated shift tolerances — the fractions must be re-read from the governing method, not copied
- Input is vendor raw data that has not yet been converted to mzML via MSConvert, so RT and m/z values per IS target are not yet available

## Inputs

- Per-IS-target measured retention time and m/z extracted from mzML files converted by MSConvert and processed via MS-DIAL
- In-run average retention time for each internal standard across the specimen sequence
- Library (.msp) RT and m/z reference values and their configured shift-from-library thresholds
- Peak-detection flag per internal standard target (detected / missing)

## Outputs

- Per-IS-target status string ('Pass', 'Warn'/'Warning', or 'Fail') for each specimen
- pytest test suite covering every threshold boundary (at, just-below, just-above each 0.66/0.80 fraction) and the missing-IS fail case

## How to apply

Read the paper's specified fractions (66% of the RT shift-from-library threshold, 80% of the in-run average RT threshold, 80% of the m/z shift-from-library threshold) from the methods text and encode them as a pure Python decision function in the rapidqcms package's rule module, taking measured RT, in-run RT, m/z, and the library thresholds as arguments. Mark an IS target 'Fail' when no peak was detected; assign 'Warning' when ΔRT > 0.66× the library RT-shift threshold, in-run ΔRT > 0.80× the in-run average threshold, or Δm/z > 0.80× the library m/z-shift threshold; otherwise 'Pass'. Write pytest unit tests covering each boundary at, just-below, and just-above each 0.66/0.80 fraction, plus the missing-IS case. The rationale is to make the paper's warning-fraction behavior reproducible and regression-safe inside the open-source Rapid QC-MS pipeline, which processes mzML converted from vendor raw files via MSConvert.

## Related tools

- **Rapid QC-MS** (Hosts the rule module in which the pure IS-status decision function is implemented; the skill reproduces its per-target 'Fail'/'Warn'/'Pass' logic) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Python** (Language for the pure decision function and pytest boundary tests; Rapid QC-MS requires Python 3.8–3.11)
- **MS-DIAL** (Supplies peak detection and identification (RT, m/z) for internal-standard targets upstream of the threshold logic)

## Examples

```
py -m pip install rapid-qc-ms; rapidqcms
```

## Evaluation signals

- Every boundary test case matches the paper's stated fractions (0.66 and 0.80) with no tolerance drift
- A missing-IS case (no detected peak) always returns 'Fail' regardless of RT or m/z values
- A target just below 66% of the library RT-shift threshold returns 'Pass' while just above returns 'Warn'
- In-run ΔRT and Δm/z boundary tests switch status exactly at 0.80× their respective thresholds
- Full pytest run passes with zero failures across all rule cases

## Limitations

- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files; other vendor formats may behave unexpectedly
- The 66%/80% fractions are the paper's specific warning-fraction choices and may not transfer to other untargeted metabolomics workflows without recalibration
- A 'Warn' status can result from any one of three independent conditions (library RT shift, in-run average RT shift, or m/z shift), so the status alone does not identify which drift occurred

## Evidence

- [other] In Rapid QC-MS, missing IS are marked 'Fail'; an individual IS receives 'Warning' if ΔRT is greater than 66% of the RT shift-from-library threshold, in-run ΔRT is greater than 80% of the in-run average threshold, or Δm/z is greater than 80% of the m/z shift-from-library threshold.: [evidence span withheld — anchor status: not_found]
