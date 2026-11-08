---
name: numerical-boundary-condition-testing
description: Use when a normalized score (e.g., a separation efficiency encoded from
  retention-time intervals) must be proven bounded before it is fed into downstream
  modeling such as Gaussian-process Bayesian optimization.
when_to_use_negative:
- Input is raw LC-MS data (mzML/mzXML) rather than an already-encoded numeric score
  — signal extraction and top-signal finding happen upstream via findTopSignals
- You need to optimize the gradient itself (acquisition function / gpModel step) rather
  than verify that the encoding of a given gradient is bounded
- The metric's boundary conditions cannot be stated analytically (no known worst/best
  case), since the derivation of GSI=0 and GSI=1 anchors is the core of the method
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: bago
  role: Provides the sepEfficiency(rtSeq, rtRange) function whose SQRTI-based GSI
    output is the object of the boundary-condition tests
  repo: https://github.com/HuanLab/bago
- name: Python
  role: Language used to implement SQRTI summation, the normalization anchors, the
    randomized boundedness checks, and the CSV logging
  repo: https://github.com/HuanLab/bago
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:75a727ae6eb883eba50bfc221b007dbe7f06c355f91ab6a26894ede7fc978083
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/huaxuyu/bago
  repo_url: https://github.com/HuanLab/bago
  tools:
  - BAGO
  - Python
  - bago
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Numerical boundary condition testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A verification skill for composite numerical encodings: derive analytic extreme-case values (co-elution and perfect separation) of a metric such as BAGO's global separation index (GSI), then computationally confirm the encoded output stays within its declared [0,1] bounds.

## When to use

Use when a normalized score (e.g., a separation efficiency encoded from retention-time intervals) must be proven bounded before it is fed into downstream modeling such as Gaussian-process Bayesian optimization.

## When NOT to use

- Input is raw LC-MS data (mzML/mzXML) rather than an already-encoded numeric score — signal extraction and top-signal finding happen upstream via findTopSignals
- You need to optimize the gradient itself (acquisition function / gpModel step) rather than verify that the encoding of a given gradient is bounded
- The metric's boundary conditions cannot be stated analytically (no known worst/best case), since the derivation of GSI=0 and GSI=1 anchors is the core of the method

## Inputs

- Retention-time vector of top MS1 signals (rtSeq, minutes)
- Gradient start/end times (rtRange, minutes)
- Boundary test vectors: a single co-eluted cluster and equally spaced peaks across rtRange
- Random retention-time configurations for boundedness sampling

## Outputs

- Float separation-efficiency / GSI value in [0,1] per test vector
- Raw per-case SQRTI values (sum of squared adjacent retention-time intervals)
- gsi_encoding_verification.csv with columns for SQRTI, normalized GSI, and pass/fail boundedness flags

## How to apply

First implement the raw metric: for BAGO's sepEfficiency(rtSeq, rtRange), sort the top-signal retention times, square the intervals between adjacent signals, and sum them to obtain SQRTI. Then derive the two analytic boundary values of the normalization: the co-elution case (all signals at one time point, one dominant interval spanning rtRange) must map to GSI = 0, and the equally spaced case (each interval equal to rtRange/(N-1), which minimizes the sum of squared intervals) must map to GSI = 1. Write a test harness that evaluates the metric on these two boundary vectors plus randomized retention-time configurations, asserting the no-separation case yields exactly 0, the equally spaced case yields exactly 1, and random cases yield values strictly inside (0,1). Write per-case raw SQRTI values, normalized GSI values, and pass/fail boundedness flags to gsi_encoding_verification.csv so failures are auditable. The rationale is that a singular 0-1 encoded value is a prerequisite input to the Gaussian process regression model, so any out-of-range or mis-anchored encoding would corrupt gradient optimization.

## Related tools

- **bago** (Provides the sepEfficiency(rtSeq, rtRange) function whose SQRTI-based GSI output is the object of the boundary-condition tests) — https://github.com/HuanLab/bago
- **Python** (Language used to implement SQRTI summation, the normalization anchors, the randomized boundedness checks, and the CSV logging) — https://github.com/HuanLab/bago

## Examples

```
eff = sepEfficiency(rtSeq, rtRange)  # assert 0 <= eff <= 1; GSI=0 for co-eluted, GSI=1 for equally spaced peaks
```

## Evaluation signals

- A single co-eluted cluster (all top signals at one time point) returns GSI exactly 0
- Equally spaced peaks with intervals of rtRange/(N-1) return GSI exactly 1
- All randomized retention-time configurations yield 0 < GSI < 1; no NaN/inf or negative values appear
- gsi_encoding_verification.csv contains one row per test case with a numeric SQRTI, a normalized GSI in [0,1], and a boundedness flag
- GSI is a single scalar per gradient run, as required when it is later fit into the Gaussian process regression model

## Limitations

- The normalization anchors assume the co-elution and equally spaced configurations are the true minima/maxima of SQRTI on the rtRange interval; if signals lie outside rtRange or N-1 <= 0, the analytic bounds do not hold
- BAGO's encoding depends on quality of the upstream top-signal selection (apexes of unique compounds); duplicate m/z features could distort the tested intervals
- Validation here confirms boundedness only for the tested vectors, not correctness of the metric against experimental separation quality

## Evidence
