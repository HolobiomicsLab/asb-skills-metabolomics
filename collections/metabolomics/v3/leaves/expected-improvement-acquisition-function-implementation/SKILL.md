---
name: expected-improvement-acquisition-function-implementation
description: Use when reconstructing or extending a Bayesian optimization loop that
  must pick the next experiment (e.g., an LC-MS gradient) by argmax of Expected Improvement
  computed from a fitted GP posterior mean and standard deviation.
when_to_use_negative:
- When the acquisition target is the second gradient rather than subsequent ones —
  BAGO computes that with computeSecondGradient(parameters, model) instead of the
  EI-based computeNextGradient rule.
- When candidate experiments are not graded by a continuous separation-efficiency
  output from LC-MS retention-time data; EI over a Matern-3/2 GPR assumes such a regression
  target.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: scikit-learn
  role: Provides the GaussianProcessRegressor with a Matern32 kernel used as the GP
    surrogate fitted to encoded gradients and separation-efficiency outputs.
- name: BAGO
  role: Python package supplying gpModel.fit(X, y), genSearchSpace, and computeNextGradient(acqFunc='ei')
    that constitute the decision rule being reconstructed.
  repo: https://github.com/HuanLab/bago
- name: Python
  role: Implementation and validation environment for the closed-form EI and the end-to-end
    argmax selection.
  repo: https://github.com/HuanLab/bago
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:98f724a6a4f086844df20baf110abef8eca17b52d97269cfac2cad0ea7de9def
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# expected-improvement-acquisition-function-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement the closed-form Expected Improvement (EI) acquisition function used by BAGO's Bayesian optimization decision rule, which selects the next LC gradient to run by maximizing EI over a candidate gradient search space using a Matern-3/2 Gaussian process surrogate of separation efficiency.

## When to use

Use when reconstructing or extending a Bayesian optimization loop that must pick the next experiment (e.g., an LC-MS gradient) by argmax of Expected Improvement computed from a fitted GP posterior mean and standard deviation.

## When NOT to use

- When the acquisition target is the second gradient rather than subsequent ones — BAGO computes that with computeSecondGradient(parameters, model) instead of the EI-based computeNextGradient rule.
- When candidate experiments are not graded by a continuous separation-efficiency output from LC-MS retention-time data; EI over a Matern-3/2 GPR assumes such a regression target.

## Inputs

- Encoded LC-gradient inputs X (encode-then-scaled gradient parameters)
- Separation-efficiency outputs y (omics-scale gradient performance metric)
- Gaussian posterior test cases (posterior mean, standard deviation, current best observed value; delta = 0.01)
- Candidate gradient search space from genSearchSpace

## Outputs

- Expected Improvement value per candidate gradient
- Per-test-case deviation between implemented and closed-form EI plus maximum absolute deviation
- Selected next gradient xnext (argmax of EI over the search space)

## How to apply

Fit a scikit-learn GaussianProcessRegressor with a Matern32 kernel to encoded LC-gradient inputs X and separation-efficiency outputs y via the gpModel.fit(X, y) interface. Generate the candidate gradient search space with genSearchSpace, mirroring BAGO's encode-then-scale preprocessing of gradients. Implement the closed-form EI from the posterior mean and standard deviation relative to the current best observed value, using exploration parameter delta = 0.01. Validate by comparing the implemented EI against the closed-form EI on controlled Gaussian posterior test cases (varying posterior mean, standard deviation, and current best value) within floating-point tolerance, then run the full decision rule end-to-end via computeNextGradient with acqFunc='ei' (the BAGO default) to select the next gradient by argmax of EI over the search space.

## Related tools

- **scikit-learn** (Provides the GaussianProcessRegressor with a Matern32 kernel used as the GP surrogate fitted to encoded gradients and separation-efficiency outputs.)
- **BAGO** (Python package supplying gpModel.fit(X, y), genSearchSpace, and computeNextGradient(acqFunc='ei') that constitute the decision rule being reconstructed.) — https://github.com/HuanLab/bago
- **Python** (Implementation and validation environment for the closed-form EI and the end-to-end argmax selection.) — https://github.com/HuanLab/bago

## Examples

```
gpModel.fit(X, y); xnext = computeNextGradient(acqFunc='ei')
```

## Evaluation signals

- Implemented EI matches the closed-form EI on every controlled Gaussian posterior test case within floating-point tolerance (max absolute deviation recorded per case).
- computeNextGradient with acqFunc='ei' runs to completion without error and returns a gradient from the genSearchSpace candidate set.
- Selected gradient is the argmax of EI over gridX — no candidate outside the search space is ever returned.
- Posterior mean and standard deviation passed into the EI formula come from the fitted GaussianProcessRegressor (Matern32 kernel), not raw observations.
- Exploration parameter delta is fixed at 0.01 in the EI formula, matching the BAGO default.

## Limitations

- BAGO's decision rule is specialized to LC gradient optimization for MS-based small molecule analysis; the EI rule is only meaningful over the encoded/scaled gradient search space BAGO defines.
- Finding the optimal gradient within ten sample analyses is the demonstrated operating regime (from over 100,000 plausible gradients); behavior outside this budget is not characterized here.
- EI operates on the separation-efficiency scalar; it does not directly consume raw mzML/mzXML data, which must first be processed (MS1/MS2 extraction, top-signal finding, sepEfficiency).

## Evidence

- [methods] computeNextGradient signature: ".. function:: computeNextGradient(self, acqFunc="ei")"
- [methods] GP surrogate: "a :class:`sklearn.gaussian_process.GaussianProcessRegressor` object."
