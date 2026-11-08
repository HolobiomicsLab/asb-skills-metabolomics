---
name: lc-gradient-encoding-search-space
description: 'Use when defining the candidate gradient configurations for an LC-MS
  gradient optimization round: you need a parameterized, constraint-filtered grid
  of mobile-phase gradients before fitting the GPR surrogate.'
when_to_use_negative:
- You only need the next gradient decision rule (fit GPR with Matern32 kernel and
  argmax EI) — that is computeNextGradient's role, downstream of this space.
- Gradients are already enumerated and encoded (gridX/scaledX exist) — regenerating
  the space would duplicate work and inflate run time.
- The task is evaluating gradient performance (separation efficiency from retention
  times) rather than defining the gradient candidate set.
tools:
- name: BAGO
  role: Provides the gpModel class whose genSearchSpace(parameters) method enumerates
    the constrained gradient search space and getMobilePhasePct validates mobile-phase
    percentages
  repo: https://github.com/HuanLab/bago
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:743a94e799354a9a74b995b2ae38ea985152566be0beb8fb6cbf804f3c0ec23c
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
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lc-gradient-encoding-search-space

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates BAGO's constrained search space of plausible LC elution gradients by enumerating p-dimensional vectors of strong mobile-phase percentages over gradient time points, enforcing monotonicity and bounded total eluting power. Produces the gridX, scaledX, and gradientPct arrays that feed the Gaussian-process surrogate and acquisition-function stages.

## When to use

Use when defining the candidate gradient configurations for an LC-MS gradient optimization round: you need a parameterized, constraint-filtered grid of mobile-phase gradients before fitting the GPR surrogate.

## When NOT to use

- You only need the next gradient decision rule (fit GPR with Matern32 kernel and argmax EI) — that is computeNextGradient's role, downstream of this space.
- Gradients are already enumerated and encoded (gridX/scaledX exist) — regenerating the space would duplicate work and inflate run time.
- The task is evaluating gradient performance (separation efficiency from retention times) rather than defining the gradient candidate set.

## Inputs

- BAGO parameter dictionary with gradient dimensionality, gradient time points, and mobile-phase settings
- LC column / mobile-phase constraints defining the reasonable total eluting-power range

## Outputs

- gridX array of enumerated gradient vectors
- scaledX array of encode-then-scaled gradient vectors
- gradientPct array of strong mobile-phase percentage per gradient and time point
- optional tabular export (e.g. search_space.csv) of the search-space arrays

## How to apply

Read gradient-definition parameters (gradient dimensionality, time points, mobile-phase settings) from the BAGO parameter dictionary via the gpModel.genSearchSpace interface. Enumerate candidate gradients as p-dimensional vectors of strong mobile-phase percentages over the gradient time points, applying two constraints during enumeration: strong mobile phase must increase monotonically along the gradient, and total eluting power must stay within a reasonable range. Use getMobilePhasePct(gradient, timePoints) to compute the strong-mobile-phase percentage at each time point as a check of constraint compliance. Emit the resulting gridX (raw grid), scaledX (encode-then-scale preprocessing mirroring BAGO's gradient encoding), and gradientPct arrays; for one dataset the enumerated space reached 261,484 gradients, consistent with the reported 'over 100,000 plausible gradients'. This space is the candidate set over which computeNextGradient later argmaxes the Expected Improvement acquisition function.

## Related tools

- **BAGO** (Provides the gpModel class whose genSearchSpace(parameters) method enumerates the constrained gradient search space and getMobilePhasePct validates mobile-phase percentages) — https://github.com/HuanLab/bago

## Examples

```
model = gpModel(parameters); gridX, scaledX = model.genSearchSpace(parameters)
```

## Evaluation signals

- Every emitted gradient row is monotonically increasing in strong mobile-phase percentage across time points.
- Every emitted gradient's total eluting power falls within the stated reasonable range.
- Reported search-space cardinality matches expected magnitudes (over 100,000 gradients; 261,484 for one dataset per supplementary parameter settings).
- getMobilePhasePct recomputation per gradient/time-point vector agrees with gradientPct entries.
- Downstream computeNextGradient(acqFunc='ei') runs to completion over the generated grid without error.

## Limitations

- The two constraints (monotonic strong mobile phase, bounded total eluting power) restrict the space to plausible gradients only; exotic non-monotonic profiles are excluded by design.
- Exact cardinality and parameter settings require expert review against the supplementary configuration; they are not fixed in the method text.
- The Windows software and Python package target LC-MS small-molecule gradient optimization; extension to other platforms is possible but out of scope of the default pipeline.

## Evidence
