---
name: retention-time-interval-encoding
description: Use when converting the retention times of top MS1 signals (e.g., from
  BAGO's findTopSignals on mzML/mzXML data) into a bounded separation-efficiency score
  to serve as the objective for LC gradient optimization.
when_to_use_negative:
- The input is already a separation-efficiency or GSI score — this skill computes
  the encoding, it does not re-normalize existing scores
- The goal is to parse raw mzML/mzXML files or extract MS1/MS2 spectra — use BAGO's
  pyopenms-based reading and findTopSignals steps first; this skill starts from the
  resulting retention times
- The evaluation target is tandem MS spectral coverage or quantification criteria
  rather than compound separation efficiency across the gradient
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3365
tools:
- name: BAGO
  role: Provides the sepEfficiency(rtSeq, rtRange) function that returns separation
    efficiency as a float, and the findTopSignals step that supplies the retention
    times of top signals
  repo: https://github.com/HuanLab/bago
- name: Python
  role: Implementation language for the SQRTI computation, GSI normalization, and
    boundedness verification; the BAGO package and analysis code are distributed in
    Python
  repo: https://github.com/HuanLab/bago
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:b2c5172104861a202c43e45aef6d7ea1209c78e2c5f88a0183759218db94562d
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# retention-time-interval-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes LC-MS compound separation performance into a bounded numeric score (GSI) by computing SQRTI — the sum of squared retention-time intervals between adjacent top MS1 signals — and normalizing it to [0,1]. This provides the singular-valued separation-efficiency objective that BAGO's Bayesian optimization of LC gradients maximizes.

## When to use

Use when converting the retention times of top MS1 signals (e.g., from BAGO's findTopSignals on mzML/mzXML data) into a bounded separation-efficiency score to serve as the objective for LC gradient optimization.

## When NOT to use

- The input is already a separation-efficiency or GSI score — this skill computes the encoding, it does not re-normalize existing scores
- The goal is to parse raw mzML/mzXML files or extract MS1/MS2 spectra — use BAGO's pyopenms-based reading and findTopSignals steps first; this skill starts from the resulting retention times
- The evaluation target is tandem MS spectral coverage or quantification criteria rather than compound separation efficiency across the gradient

## Inputs

- Retention-time vector of top MS1 signals (rtSeq, minutes), e.g., peak apexes of unique compounds from BAGO's findTopSignals
- Gradient start and end times (rtRange, in minutes)
- Optionally, raw LC-MS data in mzML or mzXML format from which top signals are extracted

## Outputs

- A singular separation-efficiency value (GSI) in [0,1] per gradient run
- Per-case SQRTI values (sum of squared retention-time intervals)
- gsi_encoding_verification.csv containing SQRTI, normalized GSI, and pass/fail boundedness flags per test case

## How to apply

Build a retention-time vector (rtSeq) of top MS1 signal apexes and the gradient start/end times (rtRange, in minutes), matching the signature of bago's sepEfficiency(rtSeq, rtRange) function which returns the separation efficiency as a float. Sort the top-signal retention times and compute SQRTI as the sum of squared intervals between adjacent signals. Normalize SQRTI to the GSI scale using the analytic boundaries: the co-elution case (all signals at one time point, one dominant interval spanning rtRange) defines GSI = 0 (no separation), and the equally spaced case (intervals equal to rtRange/(N-1), minimizing the sum of squares) defines GSI = 1 (complete separation), keeping the encoded value a singular value in 0-1. Verify boundedness computationally: a single co-eluted cluster must yield GSI = 0, equally spaced peaks across rtRange must yield GSI = 1, and random retention-time configurations must yield 0 < GSI < 1. Write the per-case SQRTI values, normalized GSI values, and pass/fail boundedness flags to gsi_encoding_verification.csv.

## Related tools

- **BAGO** (Provides the sepEfficiency(rtSeq, rtRange) function that returns separation efficiency as a float, and the findTopSignals step that supplies the retention times of top signals) — https://github.com/HuanLab/bago
- **Python** (Implementation language for the SQRTI computation, GSI normalization, and boundedness verification; the BAGO package and analysis code are distributed in Python) — https://github.com/HuanLab/bago

## Examples

```
from bago import sepEfficiency; gsi = sepEfficiency(rtSeq, rtRange)
```

## Evaluation signals

- A single co-eluted cluster (all top signals at one retention time) yields GSI = 0 (no separation)
- Equally spaced peaks across rtRange (intervals = rtRange/(N-1)) yield GSI = 1 (complete separation)
- Random retention-time configurations yield 0 < GSI < 1, and all encoded values lie within [0,1]
- Per-case SQRTI and GSI values with pass/fail flags are written to gsi_encoding_verification.csv
- Calling bago's sepEfficiency(rtSeq, rtRange) on the same inputs returns the same float as the reimplementation

## Limitations

- The encoding uses peak apexes of top MS1 signals as proxies for unique compounds, so co-elution of unobserved or low-abundance compounds is not reflected in rtSeq
- GSI is a singular value: distinct retention-time configurations can map to the same score, so equal GSI does not guarantee identical separation landscapes
- The analytic normalization assumes the co-elution boundary (one dominant interval spanning rtRange) and the equal-spacing boundary (rtRange/(N-1) intervals); inputs must use rtRange in minutes consistent with the sepEfficiency signature
- The skill is specific to BAGO's separation-efficiency definition and does not address other gradient-performance criteria (MS2 coverage, quantification) that BAGO also evaluates

## Evidence
