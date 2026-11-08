---
name: separation-efficiency-index-normalization
description: Use when scoring LC gradient performance from MS1 peak-apex retention
  times so that separation quality becomes a single 0–1 value to maximize with Bayesian
  optimization.
when_to_use_negative:
- Input is raw LC-MS data rather than already-extracted top-signal retention times
  — run readRawData/extract MS1 and findTopSignals first (e.g., via pyopenms on mzML/mzXML).
- The goal is to choose the next gradient to run (use computeNextGradient / computeSecondGradient
  with the acquisition function, not the separation-efficiency encoding itself).
- Separation is assessed from MS2-level metrics (e.g., spectral purity or dotProd
  spectral comparisons) rather than MS1 top-signal spacing.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: BAGO
  role: Python package providing sepEfficiency(rtSeq, rtRange) and the findTopSignals
    step that supplies the retention times to encode
  repo: https://github.com/HuanLab/bago
- name: Python
  role: implementation language for the SQRTI/GSI computation and boundedness verification
  repo: https://github.com/HuanLab/bago
- name: pyopenms
  role: reads mzML/mzXML raw data and extracts MS1 spectra from which top signals
    are found
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:bdfa97aaee46a9cc5727d52e95dad8fd94418ddbb844a8542fa0e2f745c1211c
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
  - pyopenms
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# separation-efficiency-index-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements BAGO's separation-efficiency encoding: from a vector of top MS1 signal retention times (rtSeq) and gradient bounds (rtRange), compute SQRTI — the sum of squared retention-time intervals between adjacent top signals — and normalize it to a Global Separation Index (GSI) bounded in [0,1] for use as the Bayesian-optimization objective.

## When to use

Use when scoring LC gradient performance from MS1 peak-apex retention times so that separation quality becomes a single 0–1 value to maximize with Bayesian optimization.

## When NOT to use

- Input is raw LC-MS data rather than already-extracted top-signal retention times — run readRawData/extract MS1 and findTopSignals first (e.g., via pyopenms on mzML/mzXML).
- The goal is to choose the next gradient to run (use computeNextGradient / computeSecondGradient with the acquisition function, not the separation-efficiency encoding itself).
- Separation is assessed from MS2-level metrics (e.g., spectral purity or dotProd spectral comparisons) rather than MS1 top-signal spacing.

## Inputs

- rtSeq: retention-time vector of top MS1 signals (peak apexes of unique compounds), in minutes
- rtRange: gradient start and end times in minutes (e.g., from the LC gradient definition)

## Outputs

- SQRTI (sum of squared retention-time intervals between adjacent top signals)
- GSI: normalized separation efficiency, a float in [0,1]
- gsi_encoding_verification.csv with per-case SQRTI, normalized GSI, and pass/fail boundedness flags

## How to apply

Take the retention times of top signals (peak apexes of unique compounds) found by bago's findTopSignals and the gradient start/end times in minutes (rtRange), matching the signature of sepEfficiency(rtSeq, rtRange) which returns separation efficiency as a float. Sort the top-signal retention times and compute SQRTI as the sum of squared intervals between adjacent signals. Normalize to GSI using the analytic boundary cases: the co-elution limit (all signals at one time point, one interval spanning rtRange) maps to GSI = 0, and equally spaced signals (intervals of rtRange/(N-1), minimizing the sum of squares) maps to GSI = 1, keeping the encoded value a singular value between 0 and 1. Verify boundedness on test vectors — a single co-eluted cluster (GSI = 0), equally spaced peaks (GSI = 1), and random configurations (0 < GSI < 1) — before feeding GSI into the gpModel Gaussian-process regression loop.

## Related tools

- **BAGO** (Python package providing sepEfficiency(rtSeq, rtRange) and the findTopSignals step that supplies the retention times to encode) — https://github.com/HuanLab/bago
- **Python** (implementation language for the SQRTI/GSI computation and boundedness verification) — https://github.com/HuanLab/bago
- **pyopenms** (reads mzML/mzXML raw data and extracts MS1 spectra from which top signals are found)

## Examples

```
from bago.encoding import sepEfficiency; gsi = sepEfficiency(rtSeq, rtRange)  # rtSeq = sorted top-signal RTs (min), rtRange = (grad_start, grad_end)
```

## Evaluation signals

- Single co-eluted cluster (all top signals at one time point) yields GSI exactly 0 (no separation).
- Equally spaced peaks across rtRange (intervals = rtRange/(N-1)) yield GSI exactly 1 (complete separation).
- Random retention-time configurations always produce 0 < GSI < 1; every encoded value lies in [0,1].
- sepEfficiency(rtSeq, rtRange) returns a Python float (singular scalar value) for any valid input vector.
- gsi_encoding_verification.csv contains per-case SQRTI, normalized GSI, and pass/fail boundedness flags with all rows passing.

## Limitations

- GSI is defined only relative to the analytic boundary cases within the given rtRange; signals falling outside rtRange or degenerate inputs (fewer than two top signals) are not covered by the normalization.
- The encoding evaluates compound separation from MS1 top signals only; it does not by itself guarantee improved MS2 spectral coverage or quantification, which BAGO reports separately.
- BAGO is specific to LC gradient optimization for MS-based small molecule analysis; other LC-based analytical platforms require extending the strategy.

## Evidence
