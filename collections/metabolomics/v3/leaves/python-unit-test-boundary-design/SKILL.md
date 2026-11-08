---
name: python-unit-test-boundary-design
description: Use when implementing or verifying Pass/Warning/Fail status logic for
  individual IS targets in Rapid QC-MS, where each fractional threshold (66% of library
  RT-shift, 80% of in-run average RT, 80% of library m/z-shift) must be boundary-tested
  with zero tolerance drift.
when_to_use_negative:
- Input is aggregated instrument-level QC data (sensitivity, drift over months) rather
  than per-IS-target RT/m/z measurements — this skill only covers individual IS status
  assignment
- The thresholds in your deployment differ from the paper's stated fractions (66%
  of library RT-shift threshold, 80% of in-run average RT threshold, 80% of library
  m/z-shift threshold), in which case the boundary tests must be rewritten against
  your configured values
- You are testing dashboard rendering or Slack/email notification logic rather than
  the threshold decision function itself
tools:
- name: Rapid QC-MS
  role: Source package whose rule module hosts the pure decision function; its 'Fail'
    rule for missing IS and fractional Warning thresholds are the behavior under test
  repo: https://github.com/czbiohub-sf/Rapid-QC-MS
- name: Python
  role: Language for the decision function and pytest unit tests; Rapid QC-MS is compatible
    with Python 3.8–3.11
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c00786
    title: 'Rapid QC-MS: Interactive Dashboard for Synchronous Mass Spectrometry Data
      Acquisition Quality Control'
  source_content_hash: sha256:2bb881a41b4906eb53592672c4bab51791487d6455262e2c486cc26fb9d35830
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
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# python-unit-test-boundary-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes Rapid QC-MS's internal-standard Pass/Warning/Fail decision rules (66%/80% fractional thresholds and a missing-peak Fail rule) as a pure Python function and validates them with pytest boundary tests at, just-below, and just-above each cutoff.

## When to use

Use when implementing or verifying Pass/Warning/Fail status logic for individual IS targets in Rapid QC-MS, where each fractional threshold (66% of library RT-shift, 80% of in-run average RT, 80% of library m/z-shift) must be boundary-tested with zero tolerance drift.

## When NOT to use

- Input is aggregated instrument-level QC data (sensitivity, drift over months) rather than per-IS-target RT/m/z measurements — this skill only covers individual IS status assignment
- The thresholds in your deployment differ from the paper's stated fractions (66% of library RT-shift threshold, 80% of in-run average RT threshold, 80% of library m/z-shift threshold), in which case the boundary tests must be rewritten against your configured values
- You are testing dashboard rendering or Slack/email notification logic rather than the threshold decision function itself

## Inputs

- Measured RT of an individual IS target
- In-run RT (in-run average) for the IS target
- Measured m/z of the IS target
- Library RT shift-from-library threshold
- In-run average RT threshold
- Library m/z shift-from-library threshold
- Peak-detection presence flag (whether the IS target has a detected peak)

## Outputs

- Status string per IS target: 'Pass', 'Warning', or 'Fail'
- pytest test suite results confirming boundary behavior for each 0.66/0.80 fraction and the missing-IS Fail case

## How to apply

First read the paper's stated threshold fractions from the methods text: an individual IS receives 'Warning' if ΔRT exceeds 66% of the RT shift-from-library threshold, in-run ΔRT exceeds 80% of the in-run average threshold, or Δm/z exceeds 80% of the m/z shift-from-library threshold. Encode these as a pure Python decision function in the rapidqcms package's rule module, taking measured RT, in-run RT, m/z, and library thresholds as arguments so comparison logic has no hidden state. Add an unconditional 'Fail' rule that triggers when an IS target has no detected peak (missing IS), which overrides Warning/Pass evaluation. Write pytest unit tests covering each boundary at, just-below, and just-above each 0.66/0.80 fraction, plus the missing-IS case, then run pytest and confirm every boundary matches the paper's stated fractions exactly with no tolerance drift.

## Related tools

- **Rapid QC-MS** (Source package whose rule module hosts the pure decision function; its 'Fail' rule for missing IS and fractional Warning thresholds are the behavior under test) — https://github.com/czbiohub-sf/Rapid-QC-MS
- **Python** (Language for the decision function and pytest unit tests; Rapid QC-MS is compatible with Python 3.8–3.11)

## Examples

```
pytest -v
```

## Evaluation signals

- At exactly 66% of the RT shift-from-library threshold the boundary test confirms whether 'Warning' triggers per the paper's 'greater than' semantics
- Just-below and just-above cases for the 66% and 80% fractions return the expected 'Pass'/'Warning' statuses with no tolerance drift (exact 0.66 and 0.80 comparisons)
- The missing-IS case always returns 'Fail' regardless of ΔRT or Δm/z values
- All three threshold conditions (RT shift-from-library, in-run average RT, m/z shift-from-library) are tested independently and in combination
- pytest run reports zero failures across every boundary test case

## Limitations

- The thresholds encoded here mirror the paper's stated fractions (0.66 and 0.80); if the rapidqcms package's configurable defaults differ or change, the tests must be updated to match the configuration, not the paper
- Rapid QC-MS has only been tested extensively on Thermo Fisher mass spectrometers, Thermo acquisition sequences, and Thermo RAW files, so IS-target behavior for other vendors may differ
- The pure decision function assumes thresholds are passed in as arguments; integration with the live QC pipeline (mzML conversion via MSConvert, MS-DIAL peak detection) is outside this skill's scope

## Evidence

- [methods] missing IS are marked 'Fail'; an individual IS receives 'Warning' if ΔRT is greater than 66% of the RT shift-from-library threshold: [evidence span withheld — anchor status: not_found]
