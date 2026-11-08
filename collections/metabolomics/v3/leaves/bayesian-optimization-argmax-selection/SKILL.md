---
name: bayesian-optimization-argmax-selection
description: Use when a GP surrogate has already been fit to observed separation-efficiency
  outputs for encoded LC gradients and you must choose which gradient (xnext) to run
  in the next LC-MS experiment.
when_to_use_negative:
- No gradient has been observed yet (the first/second run) — BAGO routes through computeSecondGradient(parameters,
  model) instead of the EI argmax rule
- The objective is not separation efficiency computed from LC-MS retention times (this
  skill's surrogate inputs are encoded LC gradients, not general hyperparameters)
- Raw data are not mzML/mzXML LC-MS files from which MS1/MS2 spectra and retention
  times can be extracted via pyopenms
tools:
- name: BAGO
  role: Provides the gpModel class (fit, genSearchSpace) and the computeNextGradient(acqFunc='ei')
    decision rule that performs the argmax-EI gradient selection
  repo: https://github.com/HuanLab/bago
- name: scikit-learn
  role: Supplies the sklearn.gaussian_process.GaussianProcessRegressor with a Matern32
    kernel used as the surrogate being optimized over
- name: pyopenms
  role: Reads raw mzML/mzXML files to extract MS1 scans and retention times used to
    compute the separation-efficiency outputs being optimized
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:8fc49f9f4de093a515cd31b911053f49fb7da7fbb0bf064b25e2e90e39e56763
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
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# bayesian-optimization-argmax-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Select the next LC gradient to test by fitting a scikit-learn GaussianProcessRegressor (Matern-3/2 kernel) to encoded gradient–separation-efficiency data and taking the argmax of the closed-form Expected Improvement acquisition function over the encoded candidate gradient search space.

## When to use

Use when a GP surrogate has already been fit to observed separation-efficiency outputs for encoded LC gradients and you must choose which gradient (xnext) to run in the next LC-MS experiment.

## When NOT to use

- No gradient has been observed yet (the first/second run) — BAGO routes through computeSecondGradient(parameters, model) instead of the EI argmax rule
- The objective is not separation efficiency computed from LC-MS retention times (this skill's surrogate inputs are encoded LC gradients, not general hyperparameters)
- Raw data are not mzML/mzXML LC-MS files from which MS1/MS2 spectra and retention times can be extracted via pyopenms

## Inputs

- Encoded LC gradient design matrix X (encode-then-scale preprocessed gradients)
- Vector y of separation-efficiency outputs computed from retention times of raw mzML/mzXML LC-MS data
- Candidate gradient search space gridX from genSearchSpace
- Current best observed separation-efficiency value and exploration parameter delta = 0.01

## Outputs

- xnext — the single next LC gradient to run, chosen by argmax of Expected Improvement over gridX
- Per-candidate EI values over the search space (for inspection of the acquisition surface)

## How to apply

Encode observed LC gradients into X and their separation-efficiency scores into y, then call gpModel.fit(X, y), which wraps a sklearn.gaussian_process.GaussianProcessRegressor with a Matern32 kernel. Generate the candidate gradient search space with genSearchSpace(parameters), mirroring BAGO's encode-then-scale preprocessing of gradients. For each candidate, evaluate the closed-form Expected Improvement from the posterior mean and standard deviation relative to the current best observed value, with the exploration parameter delta = 0.01 (BAGO's default acquisition is acqFunc='ei'). Return the gradient at the argmax of EI as xnext via computeNextGradient(acqFunc='ei'). Note this selection rule applies once at least one observed gradient exists; for the very first/second run BAGO instead uses computeSecondGradient(parameters, model).

## Related tools

- **BAGO** (Provides the gpModel class (fit, genSearchSpace) and the computeNextGradient(acqFunc='ei') decision rule that performs the argmax-EI gradient selection) — https://github.com/HuanLab/bago
- **scikit-learn** (Supplies the sklearn.gaussian_process.GaussianProcessRegressor with a Matern32 kernel used as the surrogate being optimized over)
- **pyopenms** (Reads raw mzML/mzXML files to extract MS1 scans and retention times used to compute the separation-efficiency outputs being optimized)

## Examples

```
model.fit(X, y); gridX = model.genSearchSpace(parameters); xnext = model.computeNextGradient(acqFunc='ei')
```

## Evaluation signals

- Implemented EI must match the closed-form Expected Improvement on controlled Gaussian posterior test cases (varying posterior mean, standard deviation, current best; delta = 0.01) within floating-point tolerance — record max absolute deviation per case
- computeNextGradient(acqFunc='ei') must run to completion and return a gradient lying within the genSearchSpace gridX
- The selected gradient must equal the argmax of EI computed independently over all candidates in gridX
- Repeated invocations with the same fitted GP and search space must return the identical xnext (deterministic argmax)

## Limitations

- EI argmax selects the next gradient only after observed data exist; BAGO's computeSecondGradient handles the second run separately
- Surrogate quality depends on the Matern32 kernel fit to encoded gradients; mis-encoded gradients or a poorly scaled search space will degrade the acquisition decision
- BAGO claims optimal gradient discovery within ten sample analyses from over 100,000 plausible gradients, so the search space scale (~100,000 candidates) is an assumption of the decision rule

## Evidence

- [methods] computeNextGradient selects xnext by EI argmax over the search space: ".. function:: computeNextGradient(self, acqFunc="ei")"
- [methods] The surrogate is fit through the gpModel interface: "Function to fit the data to Gaussian process regression model."
- [methods] Surrogate object is a scikit-learn GPR: "a :class:`sklearn.gaussian_process.GaussianProcessRegressor` object."
- [methods] Search space is generated inside gpModel: "This function is implemented in the :class:`gpModel` class to generate a search space."
- [readme] BAGO's purpose as a Bayesian optimization strategy for LC gradients: "BAGO is a Bayesian optimization strategy for LC gradient optimization for MS-based small molecule analysis."
- [readme] Acquisition-function rationale is documented for efficiency: "Wonder why BAGO is efficient? Read more about [acquisition functions]"
