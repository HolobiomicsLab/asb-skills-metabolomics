---
name: survival-function-mse-estimation
description: Use when two ranked lists of features (e.g., 43,919 SEQC transcripts
  ranked by read count) need a nonparametric reproducibility assessment, and a critical
  threshold N-hat controlling mFDR at a chosen alpha (0.05 or 0.01) must be derived
  from maximum rank statistics.
when_to_use_negative:
- When a model-based alternative such as the copula mixture model is required for
  parametric dependence estimates — MaRR deliberately makes no parametric assumptions
  on dependence structures or distributions of reproducible genes
- When more than two replicate experiments must be combined in a single statistic;
  the maximum rank statistic M_g = max{Rgx, Rgy} is defined for a pair of rank lists
- When feature-level effect sizes or fold changes are needed rather than reproducibility
  of rankings — the procedure operates solely on rank permutations
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Python (NumPy/pandas implementation of the closed-form MaRR estimator)
  role: Implements the survival-function MSE scan for k-hat, the mFDR-hat(i) formula,
    and the irreproducible-only simulations used for numerical validation of the estimator
- name: maximum rank reproducibility (MaRR) procedure
  role: The statistical framework within which the survival-function MSE estimation
    step operates; this skill is the k-hat/mFDR core of MaRR
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:66b757a7520eb94530634ea0556537f209e264566dbf2f110c96592d88881e87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  - Python (NumPy/pandas implementation of the closed-form MaRR estimator)
  - maximum rank reproducibility (MaRR) procedure
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# survival-function-mse-estimation

## Summary

Implements the MaRR procedure's core estimator: it fits the change point from reproducible to irreproducible signals by minimizing the mean squared error between the empirical survival function of maximum rank statistics and the weighted limiting survival function (1-lambda)*S_lambda, yielding k-hat and mFDR-based thresholds for declaring reproducible genes.

## When to use

Use when two ranked lists of features (e.g., 43,919 SEQC transcripts ranked by read count) need a nonparametric reproducibility assessment, and a critical threshold N-hat controlling mFDR at a chosen alpha (0.05 or 0.01) must be derived from maximum rank statistics.

## When NOT to use

- When a model-based alternative such as the copula mixture model is required for parametric dependence estimates — MaRR deliberately makes no parametric assumptions on dependence structures or distributions of reproducible genes
- When more than two replicate experiments must be combined in a single statistic; the maximum rank statistic M_g = max{Rgx, Rgy} is defined for a pair of rank lists
- When feature-level effect sizes or fold changes are needed rather than reproducibility of rankings — the procedure operates solely on rank permutations

## Inputs

- Two per-feature rank vectors (e.g., 43,919 SEQC transcripts ranked by read count, ties broken by random assignment), one per replicate experiment
- Nominal error rate alpha (e.g., 0.05 or 0.01)

## Outputs

- k-hat: the estimated rank change point between reproducible and irreproducible signals
- mFDR-hat(i) curve over i = k_hat+1, ..., n
- N-hat: the maximum rank threshold at which mFDR-hat(i) <= alpha
- Set of genes with M_g <= N_hat declared reproducible

## How to apply

For each gene compute the maximum rank statistic M_g = max{Rgx, Rgy} from the two rank permutations. Estimate the empirical survival function S_n_hat(x/n) and minimize MSEn(lambda) = (n - i_lambda)^{-1} * sum_x (S_n_hat(x/n) - (1-lambda) S_lambda(x/n))^2 over the grid i = 0, 1, ..., floor(0.9n), where S_lambda(x) = 1 - (x-lambda)^2/(1-lambda)^2 for lambda <= x <= 1; the argmin returns k-hat, which intentionally under-estimates n*pi1 so that n-k-hat over-estimates the number of irreproducible genes. Then compute mFDR-hat(i) = (i - k_hat)^2 / (Q(i)(n - k_hat)) for i = k_hat+1, ..., n, where Q(i) counts genes with M_g <= i, and set N_hat = max{i : mFDR-hat(i) <= alpha}, declaring all genes with M_g <= N_hat reproducible. The procedure requires no tuning parameters; the only user choices are alpha (0.05 or 0.01) and the 0.9n scan boundary.

## Related tools

- **Python (NumPy/pandas implementation of the closed-form MaRR estimator)** (Implements the survival-function MSE scan for k-hat, the mFDR-hat(i) formula, and the irreproducible-only simulations used for numerical validation of the estimator)
- **maximum rank reproducibility (MaRR) procedure** (The statistical framework within which the survival-function MSE estimation step operates; this skill is the k-hat/mFDR core of MaRR)

## Examples

```
from marr_core import estimate_k, mFDR; k_hat = estimate_k(M_g); N_hat = mFDR(M_g, k_hat, alpha=0.01)
```

## Evaluation signals

- On simulated irreproducible-only data (two independent uniform rank permutations), the empirical probability mass function of M_g/n must match (2i-1)/n^2 from Proposition 1
- On ideal-setting data (e.g., pi1 = 0.35, n = 1000, per Figure 1, under assumptions (I1)-(I3)), pi1-hat must converge to pi1 as n grows (Theorem 2)
- At alpha = 0.01 on SEQC comparisons 1, 2, 4, 5, the resulting pi1-hat values must match the reported 0.851, 0.847, 0.900, 0.898, which exceed the copula mixture model's 0.773, 0.767, 0.861, 0.649
- The counts of reproducible transcripts at alpha = 0.01 must agree with Table 5 / Figure 9 of the article
- Sanity check that k-hat <= n*pi1 by construction, since k-hat under-estimates n*pi1

## Limitations

- k-hat under-estimates n*pi1 by design; the estimator is conservative in that n-k-hat over-estimates the true number of irreproducible genes
- The limiting survival function S_lambda holds asymptotically; the theory rests on assumptions (I1)-(I3) for the ideal-setting behavior of pi1-hat
- Ranking ties must be broken by random assignment, introducing stochasticity in the maximum rank statistics

## Evidence

- [other] The mFDR estimate uses k-hat which under-estimates n*pi1 so n-k-hat over-estimates the number of irreproducible genes; mFDR-hat(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)): [evidence span withheld — anchor status: not_found]
- [results] MaRR outperforms the copula mixture model on SEQC: pi1-hat = 0.851, 0.847, 0.900, 0.898 for comparisons 1, 2, 4, 5: [evidence span withheld — anchor status: not_found]
