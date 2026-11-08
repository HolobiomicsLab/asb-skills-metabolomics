---
name: mobile-phase-constraint-validation
description: Use when enumerating candidate LC gradients (p-dimensional mobile-phase
  percentage vectors over gradient time points) for BAGO's gpModel.genSearchSpace
  and you must filter out gradients that are not physically executable.
when_to_use_negative:
- When the goal is scoring gradients after experimentation (separation efficiency
  from retention times via sepEfficiency) rather than defining the pre-experimental
  search space
- When optimizing isocratic or non-monotonic gradient programs that intentionally
  violate the monotonic strong-mobile-phase assumption
- When the objective is selecting the next gradient to run (computeNextGradient with
  an acquisition function such as EI), which operates on the already-constrained space
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: BAGO
  role: Python package whose gpModel class implements genSearchSpace and getMobilePhasePct
    used to enumerate and constrain gradients
  repo: https://github.com/HuanLab/bago
- name: Python
  role: Runtime for the enumeration, constraint filtering, and assertion script; version
    3.8 or later
  repo: https://github.com/HuanLab/bago
- name: scikit-learn
  role: Provides the GaussianProcessRegressor downstream of the constrained space;
    the search space feeds the gpModel fitting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:06f099ae8ebe5f7adb223f7274a9aed8a69b00406bb20c1787a4603fab7c22cf
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Mobile-phase constraint validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation step in the BAGO gradient search-space generator that enforces two physical plausibility constraints — monotonically increasing strong mobile phase percentage and total eluting power within a reasonable range — on every enumerated LC gradient before it enters the Bayesian optimization search space.

## When to use

Use when enumerating candidate LC gradients (p-dimensional mobile-phase percentage vectors over gradient time points) for BAGO's gpModel.genSearchSpace and you must filter out gradients that are not physically executable.

## When NOT to use

- When the goal is scoring gradients after experimentation (separation efficiency from retention times via sepEfficiency) rather than defining the pre-experimental search space
- When optimizing isocratic or non-monotonic gradient programs that intentionally violate the monotonic strong-mobile-phase assumption
- When the objective is selecting the next gradient to run (computeNextGradient with an acquisition function such as EI), which operates on the already-constrained space

## Inputs

- BAGO parameter dictionary (gradient dimensionality, gradient time points, mobile-phase settings) as passed to gpModel.genSearchSpace
- Enumerated candidate gradients as p-dimensional vectors of strong mobile-phase percentages over gradient time points

## Outputs

- Constrained search-space arrays gridX, scaledX, and gradientPct (strong mobile-phase percentage per gradient and time point), e.g. written to search_space.csv
- Validation script result asserting every emitted row is monotonic in strong mobile phase and within the stated eluting-power range

## How to apply

During enumeration of candidate gradients by gpModel.genSearchSpace, represent each gradient as a p-dimensional vector of strong mobile-phase percentages over the gradient time points and compute its strong-mobile-phase composition with getMobilePhasePct(gradient, timePoints). Reject any candidate whose strong mobile phase percentage is not monotonically increasing along the gradient, and reject any candidate whose total eluting power falls outside the reasonable range defined in the BAGO parameter dictionary. Write the surviving candidates to the gridX, scaledX, and gradientPct arrays (e.g. as search_space.csv), then run an assertion script that re-checks every emitted row for monotonicity and eluting-power range compliance. The resulting constrained space should be large — the paper reports over 100,000 plausible gradients, and 261,484 for one dataset — and its exact cardinality is accepted via expert review against the supplementary parameter settings.

## Related tools

- **BAGO** (Python package whose gpModel class implements genSearchSpace and getMobilePhasePct used to enumerate and constrain gradients) — https://github.com/HuanLab/bago
- **Python** (Runtime for the enumeration, constraint filtering, and assertion script; version 3.8 or later) — https://github.com/HuanLab/bago
- **scikit-learn** (Provides the GaussianProcessRegressor downstream of the constrained space; the search space feeds the gpModel fitting)

## Examples

```
from bago import gpModel; m = gpModel(parameters); gridX, scaledX, gradientPct = m.genSearchSpace(parameters); assert all(b > a for a, b in zip(g[:-1], g[1:]) for g in gradientPct)
```

## Evaluation signals

- Every emitted gradient row is monotonically increasing in strong mobile-phase percentage across all gradient time points
- Every emitted gradient's total eluting power falls within the reasonable range stated in the supplementary parameter settings
- getMobilePhasePct(gradient, timePoints) recomputation matches the stored gradientPct values for each candidate
- Search-space cardinality is consistent with the reported scale (over 100,000 gradients; 261,484 for one dataset) under expert_review against supplementary settings
- gridX, scaledX, and gradientPct arrays are mutually consistent in length and written to search_space.csv without dropped or duplicated candidates

## Limitations

- The 'reasonable range' of total eluting power is defined by the BAGO parameter dictionary / supplementary settings; the exact numeric bounds are not stated in the main text and must be obtained there
- Constraint checking occurs during enumeration; the exact cardinality of the space is accepted via expert_review, so a programmatic cardinality check alone is not authoritative

## Evidence

- [readme] _bago_ is a Python package for supporting the proposed Bayesian Optimization framework of LC gradient optimization.: [evidence span withheld — anchor status: external_doc]
