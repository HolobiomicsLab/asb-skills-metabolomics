---
name: gpmodel-searchspace-configuration
description: 'Use when configuring the first stage of a BAGO LC-MS gradient optimization:
  you need to enumerate the plausible-gradient search space (gridX, scaledX, gradientPct)
  before fitting the Gaussian process model.'
when_to_use_negative:
- Experimental MS1/MS2 data are already collected and you need to score gradients
  — use sepEfficiency-based evaluation instead, not search-space generation
- The Gaussian process model is already fitted and you need the next gradient to run
  — use computeNextGradient / computeSecondGradient rather than regenerating the space
tools:
- name: BAGO
  role: Python package providing the gpModel class whose genSearchSpace method enumerates
    the constrained gradient search space
  repo: https://github.com/HuanLab/bago
- name: Python 3.8
  role: Runtime environment for executing the BAGO package (Python 3.8 or later)
- name: scikit-learn
  role: Provides the GaussianProcessRegressor that the generated search space feeds
    into for the Bayesian optimization loop
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:ea2b6dbd161d5ef430b6e0d5acb43f5a92f5863a3ce5d2dc75705c6dd129ac77
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
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gpmodel-searchspace-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates the first-stage BAGO search space of plausible LC gradients by enumerating p-dimensional mobile-phase percentage vectors with gpModel.genSearchSpace, enforcing monotonic strong-solvent increase and bounded total eluting power. This constrains downstream Bayesian optimization to physically reasonable gradients (>100,000 candidates).

## When to use

Use when configuring the first stage of a BAGO LC-MS gradient optimization: you need to enumerate the plausible-gradient search space (gridX, scaledX, gradientPct) before fitting the Gaussian process model.

## When NOT to use

- Experimental MS1/MS2 data are already collected and you need to score gradients — use sepEfficiency-based evaluation instead, not search-space generation
- The Gaussian process model is already fitted and you need the next gradient to run — use computeNextGradient / computeSecondGradient rather than regenerating the space

## Inputs

- BAGO parameter dictionary with gradient dimensionality, gradient time points, and mobile-phase settings
- Gradient time-point vector for strong-mobile-phase percentage checks

## Outputs

- gridX array of enumerated gradient configurations
- scaledX array of scaled gradient configurations
- gradientPct attribute (strong mobile-phase percentage for each gradient)
- search_space.csv tabular file of the enumerated search space

## How to apply

Read the gradient-definition parameters (gradient dimensionality, time points, mobile-phase settings) from the BAGO parameter dictionary and call gpModel.genSearchSpace(parameters) to enumerate candidate gradients as p-dimensional vectors of strong mobile-phase percentages over the gradient time points, producing gridX and scaledX arrays plus the gradientPct attribute. During enumeration apply two constraints: strong mobile-phase percentage must be monotonically increasing along the gradient, and total eluting power must stay within a reasonable range, so only physically plausible gradients remain. Verify each candidate by computing getMobilePhasePct(gradient, timePoints), which derives the strong-mobile-phase percentage per time point as a constraint-compliance check. Write the resulting arrays to a tabular output (search_space.csv) for downstream Gaussian-process fitting (gpModel.fit) and acquisition-function-driven selection via computeNextGradient. The resulting space is large — over 100,000 plausible gradients (261,484 for one dataset) — which is precisely why Bayesian optimization with ~10 sample analyses, rather than exhaustive testing, is used to locate the optimum.

## Related tools

- **BAGO** (Python package providing the gpModel class whose genSearchSpace method enumerates the constrained gradient search space) — https://github.com/HuanLab/bago
- **Python 3.8** (Runtime environment for executing the BAGO package (Python 3.8 or later))
- **scikit-learn** (Provides the GaussianProcessRegressor that the generated search space feeds into for the Bayesian optimization loop)

## Examples

```
from bago import gpModel; model = gpModel(); gridX, scaledX, gradientPct = model.genSearchSpace(parameters)
```

## Evaluation signals

- Every emitted row in search_space.csv is monotonically increasing in strong mobile-phase percentage
- Every emitted row keeps total eluting power within the stated reasonable range
- getMobilePhasePct computed per gradient and time-point vector matches the gradientPct values stored for each candidate
- Search-space cardinality is consistent with the reported scale (over 100,000; 261,484 for one dataset) when using the supplementary parameter settings

## Limitations

- The exact cardinality of the search space depends on the supplementary parameter settings (gradient dimensionality, time points, mobile-phase settings); different settings yield different space sizes
- Constraints are defined as 'monotonically increasing' strong mobile phase and a 'reasonable range' of total eluting power — the concrete numeric bounds must be supplied via the parameter dictionary, not inferred
- The skill only defines the first-stage candidate space; it does not evaluate gradients (separation efficiency) or select the next experiment

## Evidence
