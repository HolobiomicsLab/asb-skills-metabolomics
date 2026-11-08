---
name: chromatographic-gradient-evaluation
description: Use when raw LC-MS (mzML/mzXML) data from a gradient run must be reduced
  to a single separation-performance number (GSI) to evaluate or optimize a chromatographic
  gradient.
when_to_use_negative:
- The task is proposing or optimizing the gradient itself via Bayesian optimization
  (computeNextGradient/genSearchSpace), not scoring the separation performance of
  a given gradient's LC-MS run.
- The goal is MS2-level quality assessment (e.g., spectral purity or dotProd similarity
  between MS2 spectra with mzTol=0.02) rather than MS1 peak-apex separation encoding.
- Retention times of top signals are unavailable — e.g., only MS2 or DDA fragment
  data exist — since SQRTI is defined solely from MS1 retention-time intervals.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: BAGO
  role: Provides the sepEfficiency(rtSeq, rtRange) encoding that computes separation
    efficiency as a float from top-signal retention times
  repo: https://github.com/HuanLab/bago
- name: pyopenms
  role: Reads mzML/mzXML raw LC-MS data and extracts MS1 scans needed to locate top
    signals feeding the separation-efficiency calculation
- name: Python 3.8
  role: Runtime environment (via pip-installable bago package) for implementing SQRTI/GSI
    computation and boundedness verification
  repo: https://github.com/HuanLab/bago
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:7feeca03e3f106c6d960df7bbcd5bdb557ea4c620973cb9d859f263ee6b7bc57
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
  - Python 3.8
  - pyopenms
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chromatographic-gradient-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores LC gradient separation performance by encoding peak apexes of unique compounds into a sum of squared retention-time intervals (SQRTI) and normalizing it to a bounded Global Separation Index (GSI) between 0 and 1, giving Bayesian optimization a single-value objective.

## When to use

Use when raw LC-MS (mzML/mzXML) data from a gradient run must be reduced to a single separation-performance number (GSI) to evaluate or optimize a chromatographic gradient.

## When NOT to use

- The task is proposing or optimizing the gradient itself via Bayesian optimization (computeNextGradient/genSearchSpace), not scoring the separation performance of a given gradient's LC-MS run.
- The goal is MS2-level quality assessment (e.g., spectral purity or dotProd similarity between MS2 spectra with mzTol=0.02) rather than MS1 peak-apex separation encoding.
- Retention times of top signals are unavailable — e.g., only MS2 or DDA fragment data exist — since SQRTI is defined solely from MS1 retention-time intervals.

## Inputs

- Raw LC-MS data in mzML or mzXML format (read via pyopenms)
- Extracted MS1 spectra as ms1Spectrum objects from which top signals (peak apexes of unique compounds) are found
- Retention-time vector of top MS1 signals (rtSeq) and gradient start/end times (rtRange, minutes)

## Outputs

- Separation efficiency as a float (SQRTI normalized to GSI in [0,1]) returned by sepEfficiency(rtSeq, rtRange)
- Per-case SQRTI values, normalized GSI values, and boundedness pass/fail flags written to gsi_encoding_verification.csv

## How to apply

Extract MS1 scans from the raw mzML/mzXML file with pyopenms, find top signals as the peak apexes of unique compounds (getUniqueMz for uniqueness, precsMzTol=0.01), and collect their retention times into an rtSeq vector together with the gradient start/end times rtRange in minutes. Compute SQRTI by sorting the top-signal retention times and summing the squared intervals between adjacent signals, matching bago's sepEfficiency(rtSeq, rtRange) signature. Normalize SQRTI to the GSI scale using analytic boundaries: the co-elution case (all signals at one time point) maps to GSI=0 (no separation) and the equally spaced case (intervals equal to rtRange/(N-1)) maps to GSI=1 (complete separation), keeping the encoded value a singular value in [0,1]. This GSI is the objective BAGO's Gaussian-process model (sklearn GaussianProcessRegressor) and acquisition function use to propose the next gradient.

## Related tools

- **BAGO** (Provides the sepEfficiency(rtSeq, rtRange) encoding that computes separation efficiency as a float from top-signal retention times) — https://github.com/HuanLab/bago
- **pyopenms** (Reads mzML/mzXML raw LC-MS data and extracts MS1 scans needed to locate top signals feeding the separation-efficiency calculation)
- **Python 3.8** (Runtime environment (via pip-installable bago package) for implementing SQRTI/GSI computation and boundedness verification) — https://github.com/HuanLab/bago

## Examples

```
from bago import sepEfficiency; gsi = sepEfficiency(rtSeq, rtRange)
```

## Evaluation signals

- All encoded GSI values must lie within [0,1]
- A single co-eluted cluster of top signals (all at one retention time) must yield GSI = 0
- Equally spaced peaks across rtRange (intervals = rtRange/(N-1)) must yield GSI = 1
- Random retention-time configurations must yield 0 < GSI < 1
- Per-case SQRTI, GSI, and pass/fail boundedness flags must be present in gsi_encoding_verification.csv

## Limitations

- GSI requires a known gradient time range (rtRange in minutes); co-elution and equal-spacing are the fixed normalization anchors, so the encoding assumes N-1 intervals over rtRange.
- The encoding depends on correctly identifying unique compounds' peak apexes (getUniqueMz with precsMzTol=0.01); noisy or overlapping MS1 apexes will distort SQRTI.
- Raw MS data underlying the paper are only available on demand, so verification must be done on synthetic test vectors unless those data are obtained.

## Evidence

- [intro] Separation efficiency was defined to evaluate the performance of a gradient.: "Separation efficiency was defined to evaluate the performance of a gradient."
- [methods] sepEfficiency signature: ".. function:: sepEfficiency(rtSeq, rtRange)"
- [methods] Calculate the separation efficiency using a series of retention times.: "Calculate the separation efficiency using a series of retention times."
- [methods] Function to find the top signals in the raw LCMS data.: "Function to find the top signals in the raw LCMS data."
- [methods] mzML and mzXML files are supported.: "mzML and mzXML files are supported. This function is achieved by using the :mod:`pyopenms` module."
- [readme] BAGO is a Bayesian optimization strategy for LC gradient optimization for MS-based small molecule analysis.: "BAGO is a Bayesian optimization strategy for LC gradient optimization for MS-based small molecule analysis."
