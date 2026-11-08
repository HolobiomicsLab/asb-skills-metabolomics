---
name: gaussian-process-surrogate-regression
description: Use when a sequential optimization loop needs a probabilistic surrogate
  fitted to small experimental data (e.g., LC gradient parameters vs. separation efficiency)
  to support acquisition-function-based selection of the next experiment.
when_to_use_negative:
- When the objective lacks a probabilistic surrogate requirement — e.g., raw LC-MS
  preprocessing (mzML/mzXML reading, MS1/MS2 spectrum extraction, top-signal finding)
  should use the pyopenms-based pipeline instead of the GP model
- When experimental data are not from LC gradient optimization against a separation-efficiency
  metric — the Matern32 kernel and encode-then-scale preprocessing are tuned to BAGO's
  gradient-encoding scheme
- When paired MS2 comparison tasks are needed (e.g., spectral similarity via dotProd
  with mzTol=0.02) — that is a separate function outside the GP surrogate
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: scikit-learn
  role: Provides the GaussianProcessRegressor class used as the surrogate model with
    a Matern32 kernel, fitted via gpModel.fit(X, y)
- name: BAGO
  role: 'Python package implementing the Bayesian optimization workflow: gpModel class,
    genSearchSpace, fit, and computeNextGradient(acqFunc=''ei'')'
  repo: https://github.com/HuanLab/bago
- name: Python
  role: Runtime environment (Python 3.8 or later) for running the surrogate fitting
    and gradient selection pipeline
  repo: https://github.com/HuanLab/bago
- name: pyopenms
  role: Used upstream to read raw LC-MS mzML/mzXML data from which separation-efficiency
    training outputs are derived
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:0ba9edbb7dff6a0d40679ab773322f44d2c4f2a53cb14fed3a7356dd59f2985d
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gaussian-process-surrogate-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fit a Gaussian process regression surrogate (scikit-learn GaussianProcessRegressor with a Matern32 kernel) to encoded LC-gradient inputs and separation-efficiency outputs, forming the probabilistic model BAGO uses to score candidate gradients in its Bayesian optimization loop.

## When to use

Use when a sequential optimization loop needs a probabilistic surrogate fitted to small experimental data (e.g., LC gradient parameters vs. separation efficiency) to support acquisition-function-based selection of the next experiment.

## When NOT to use

- When the objective lacks a probabilistic surrogate requirement — e.g., raw LC-MS preprocessing (mzML/mzXML reading, MS1/MS2 spectrum extraction, top-signal finding) should use the pyopenms-based pipeline instead of the GP model
- When experimental data are not from LC gradient optimization against a separation-efficiency metric — the Matern32 kernel and encode-then-scale preprocessing are tuned to BAGO's gradient-encoding scheme
- When paired MS2 comparison tasks are needed (e.g., spectral similarity via dotProd with mzTol=0.02) — that is a separate function outside the GP surrogate

## Inputs

- Encoded LC-gradient input matrix X (numerically encoded gradient parameters, encode-then-scaled)
- Separation-efficiency output vector y computed from MS1 retention times of LC-MS runs
- Gradient search space parameters (for genSearchSpace candidate generation)

## Outputs

- Fitted GaussianProcessRegressor surrogate (gpModel) with posterior mean and standard deviation over the search space
- Candidate gradient grid (gridX) with GP predictions for acquisition-function evaluation
- Next gradient (xnext) selected by argmax of the acquisition function

## How to apply

Encode each LC gradient into numeric input vectors X and compute the corresponding separation-efficiency values y, then fit a scikit-learn GaussianProcessRegressor with a Matern32 kernel via the BAGO gpModel.fit(X, y) interface. Generate the candidate search space with genSearchSpace, mirroring BAGO's encode-then-scale preprocessing of gradients so the surrogate predicts on the same scale as training data. The fitted GP posterior mean and standard deviation then feed closed-form acquisition functions (e.g., Expected Improvement with exploration parameter delta = 0.01), and the next gradient is selected by argmax of the acquisition over the search space via computeNextGradient(acqFunc='ei'). Validate the surrogate-driven decision rule by comparing implemented EI against closed-form EI on controlled Gaussian posterior test cases within floating-point tolerance, confirming the argmax selection runs without error.

## Related tools

- **scikit-learn** (Provides the GaussianProcessRegressor class used as the surrogate model with a Matern32 kernel, fitted via gpModel.fit(X, y))
- **BAGO** (Python package implementing the Bayesian optimization workflow: gpModel class, genSearchSpace, fit, and computeNextGradient(acqFunc='ei')) — https://github.com/HuanLab/bago
- **Python** (Runtime environment (Python 3.8 or later) for running the surrogate fitting and gradient selection pipeline) — https://github.com/HuanLab/bago
- **pyopenms** (Used upstream to read raw LC-MS mzML/mzXML data from which separation-efficiency training outputs are derived)

## Examples

```
from sklearn.gaussian_process import GaussianProcessRegressor; model = gpModel(); model.fit(X, y); model.genSearchSpace(parameters); xnext = model.computeNextGradient(acqFunc='ei')
```

## Evaluation signals

- Implemented Expected Improvement matches the closed-form EI on controlled Gaussian posterior test cases (varying posterior mean, standard deviation, and current best value; delta = 0.01) within floating-point tolerance, with maximum absolute deviation reported per case
- computeNextGradient with acqFunc='ei' (the BAGO default) runs end-to-end and returns the argmax-EI gradient from gridX without error
- Fitted gpModel predicts on the encoded-then-scaled gradient search space consistently with the training preprocessing (encode-then-scale)
- Surrogate is fitted to the correct input/output pairing: encoded LC-gradient X vs. separation-efficiency y from retention-time-derived metrics

## Limitations

- BAGO's optimization assumes separation efficiency as the objective computed from a series of retention times; other objectives require redefining the encoding and output
- Surrogate quality depends on limited experimental data (BAGO targets convergence within ~10 sample analyses from over 100,000 plausible gradients), so posterior estimates are uncertain early in the loop
- The acquisition decision rule defaults to 'ei'; deviating to other acquisition functions changes the selection behavior and was not the default path validated here

## Evidence
