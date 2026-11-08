---
name: asymptotic-consistency-validation
description: Use when implementing or verifying the MaRR procedure's k-hat threshold
  and mFDR estimation and need to confirm consistency (pi1-hat -> pi1) and correct
  irreproducible-null distribution assumptions.
when_to_use_negative:
- The goal is a parametric or model-based reproducibility analysis with assumptions
  on dependence structures — MaRR is explicitly nonparametric and requires no tuning
  parameters
- 'The input is observed replicate rank data for real genes rather than simulated
  data: this skill only validates estimator behavior on controlled irreproducible-only
  and ideal-setting simulations, not on SEQC-style experimental data'
- The number of genes n is too small for the limiting survival function S_lambda(x)
  = 1 - (x-lambda)^2/(1-lambda)^2 to approximate S_n_hat, invalidating the asymptotic
  grid scan
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Python
  role: Implementation language for the MaRR core routine (k-hat estimation and mFDR
    step) and the validation simulations
- name: NumPy
  role: Vectorized computation of rank statistics M_g, empirical survival function
    S_n_hat, and the MSEn(lambda) grid scan
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:c10f8688fe625beee138b784ea8433f511a099a5477a7c2d5da055188f2a2863
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# asymptotic-consistency-validation

## Summary

Validates that the MaRR k-hat estimator (minimizer of MSEn(lambda) between the empirical survival function S_n_hat and the limiting survival S_lambda) and the derived mFDR-hat(i) estimates behave as the asymptotic theory predicts, using simulated irreproducible-only and ideal-setting rank data.

## When to use

Use when implementing or verifying the MaRR procedure's k-hat threshold and mFDR estimation and need to confirm consistency (pi1-hat -> pi1) and correct irreproducible-null distribution assumptions.

## When NOT to use

- The goal is a parametric or model-based reproducibility analysis with assumptions on dependence structures — MaRR is explicitly nonparametric and requires no tuning parameters
- The input is observed replicate rank data for real genes rather than simulated data: this skill only validates estimator behavior on controlled irreproducible-only and ideal-setting simulations, not on SEQC-style experimental data
- The number of genes n is too small for the limiting survival function S_lambda(x) = 1 - (x-lambda)^2/(1-lambda)^2 to approximate S_n_hat, invalidating the asymptotic grid scan

## Inputs

- Simulated irreproducible-only rank data (two independent uniform rank permutations for n genes, maximum rank statistic M_g per Definition 1)
- Ideal-setting rank data generated under assumptions (I1)-(I3), e.g. pi1 = 0.35, n = 1000 (per Figure 1)
- Grid of candidate thresholds i = 0, 1, ..., floor(0.9n)
- Nominal FDR levels alpha = 0.05 and 0.01

## Outputs

- k_hat = argmin_i MSEn(i/n) from the survival-function MSE scan
- mFDR-hat(i) = (i - k_hat)^2 / (Q(i)(n - k_hat)) for i = k_hat+1, ..., n
- Selected set size N_hat = max{i : mFDR-hat(i) <= alpha}
- validation_report.json recording PMF match to (2i-1)/n^2 and pi1-hat convergence results
- Standalone implementation saved as marr_core.py

## How to apply

Simulate irreproducible-only datasets by drawing two independent uniform rank permutations for n genes and computing M_g = max{Rgx, Rgy}; check that the empirical PMF of M_g/n matches (2i-1)/n^2 from Proposition 1. Compute the empirical survival function S_n_hat(x/n) and the theoretical S_lambda(x) = 1 - (x-lambda)^2/(1-lambda)^2 for lambda <= x <= 1, then minimize MSEn(lambda) = (n - i_lambda)^{-1} * sum_x (S_n_hat(x/n) - (1-lambda) S_lambda(x/n))^2 over the grid i = 0,...,floor(0.9n) to obtain k_hat. Form mFDR-hat(i) = (i - k_hat)^2 / (Q(i)(n - k_hat)) for i = k_hat+1,...,n with Q(i) counting genes with M_g <= i, and select N_hat = max{i : mFDR-hat(i) <= alpha} at alpha = 0.05 and 0.01. Finally generate ideal-setting data (e.g. pi1 = 0.35, n = 1000, per Figure 1) under assumptions (I1)-(I3) to confirm pi1-hat converges to pi1 as n grows (Theorem 2). The rationale is that k_hat under-estimates n*pi1, so n - k_hat over-estimates the irreproducible count, making the mFDR bound conservative.

## Related tools

- **Python** (Implementation language for the MaRR core routine (k-hat estimation and mFDR step) and the validation simulations)
- **NumPy** (Vectorized computation of rank statistics M_g, empirical survival function S_n_hat, and the MSEn(lambda) grid scan)

## Examples

```
from marr_core import estimate_k, mfdr; k_hat = estimate_k(S_n_hat, n); N_hat = mfdr(k_hat, Q, alpha=0.05)  # results logged to validation_report.json
```

## Evaluation signals

- Empirical PMF of M_g/n on irreproducible-only data matches (2i-1)/n^2 from Proposition 1
- In ideal-setting simulations (e.g. pi1 = 0.35, n = 1000 under assumptions (I1)-(I3)), pi1-hat converges to pi1 as n grows (Theorem 2)
- k_hat falls at or below n*pi1 (under-estimate), consistent with n - k_hat over-estimating the number of irreproducible genes
- mFDR-hat(i) is non-decreasing in i and N_hat = max{i : mFDR-hat(i) <= alpha} exists for alpha = 0.05 and 0.01
- MSEn(i/n) scan is restricted to i = 0,...,floor(0.9n) and returns a unique argmin

## Limitations

- k_hat under-estimates n*pi1 by design, so n - k_hat over-estimates the true number of irreproducible genes; declared reproducible sets are conservative
- Validation relies on the limiting survival function S_lambda(x) = 1 - (x-lambda)^2/(1-lambda)^2 valid for lambda <= x <= 1; behavior outside that range or for small n is not guaranteed
- Asymptotic convergence of pi1-hat is established only under assumptions (I1)-(I3) in ideal settings; deviations from these assumptions are not covered by the validation

## Evidence

- [methods] pmf_prop1: [evidence span withheld — anchor status: not_found]
- [methods] mse_scan: [evidence span withheld — anchor status: not_found]
- [methods] mfdr_step: [evidence span withheld — anchor status: not_found]
- [results] underestimate: [evidence span withheld — anchor status: not_found]
- [intro] objective: "Based on a maximum rank statistic, our procedure identifies where the change from reproducible to irreproducible signals begins by minimizing the mean squared error between observed and theoretical"
