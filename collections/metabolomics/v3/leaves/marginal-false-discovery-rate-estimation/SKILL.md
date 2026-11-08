---
name: marginal-false-discovery-rate-estimation
description: Use when you have two (or three) replicate rank lists over the same n
  genes and need an error-controlled call set of reproducible signals without parametric
  assumptions on the dependence structure of reproducible genes.
when_to_use_negative:
- You want a parametric model of the dependence structure among reproducible signals
  (e.g. the copula mixture model of Li et al. 2011) rather than a nonparametric FDR-controlled
  call set — the copula model fits dependence via (mu_A, rho, pi1), while MaRR makes
  no parametric assumptions.
- There is only one experiment / no replicate rank lists to compare, so no maximum
  rank statistic M_g can be formed.
- You need a decision boundary sensitive to signals far off the diagonal y=x; note
  the MaRR region {M_g <= N-hat} is square and, in the near-perfect-correlation setting
  (r0 = 0.95, pi1 = 0.80), the copula mixture model instead captures tail-shaped regions
  — choose the method matching the boundary shape you want.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Python (NumPy)
  role: Implement the MSEn(lambda) minimization for k-hat, the Q(i) cumulative counts,
    and the mFDR-hat(i) estimator; the closed-form MaRR estimator is implemented in
    NumPy/pandas for the SEQC analysis.
- name: maximum rank reproducibility (MaRR) procedure
  role: 'The method this skill implements: k-hat estimation plus mFDR-based thresholding
    on the maximum rank statistic.'
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  - task_004
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:a067cad15c568450972b71b9294204b90a67c4314e2541d91e9145bfe0d99e90
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  - Python (NumPy)
  - Python (NumPy/pandas implementation of the closed-form MaRR estimator)
  - SciPy
  - maximum rank reproducibility (MaRR) procedure
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# marginal-false-discovery-rate-estimation

## Summary

Computes the marginal false discovery rate mFDR-hat(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) from the maximum rank statistic M_g = max{Rgx, Rgy} in the MaRR procedure, and selects the threshold N-hat = max{i : mFDR-hat(i) <= alpha} so genes with M_g <= N-hat are declared reproducible at nominal alpha (0.05 or 0.01) with no tuning parameters.

## When to use

Use when you have two (or three) replicate rank lists over the same n genes and need an error-controlled call set of reproducible signals without parametric assumptions on the dependence structure of reproducible genes.

## When NOT to use

- You want a parametric model of the dependence structure among reproducible signals (e.g. the copula mixture model of Li et al. 2011) rather than a nonparametric FDR-controlled call set — the copula model fits dependence via (mu_A, rho, pi1), while MaRR makes no parametric assumptions.
- There is only one experiment / no replicate rank lists to compare, so no maximum rank statistic M_g can be formed.
- You need a decision boundary sensitive to signals far off the diagonal y=x; note the MaRR region {M_g <= N-hat} is square and, in the near-perfect-correlation setting (r0 = 0.95, pi1 = 0.80), the copula mixture model instead captures tail-shaped regions — choose the method matching the boundary shape you want.

## Inputs

- Two replicate rank lists (Rgx, Rgy) over the same n entities (e.g. 43,919 SEQC transcripts ranked by read count, rank 1 = most reads, ties broken randomly)
- Maximum rank statistics M_g = max{Rgx, Rgy} for every gene (Definition 1)
- Nominal FDR level alpha (article uses 0.05 and 0.01, e.g. alpha = 0.01 for the six SEQC comparisons)

## Outputs

- k-hat, the estimated change point where irreproducible signals begin, minimizing MSEn(i/n) over i = 0, ..., floor(0.9n)
- pi1-hat = k-hat/n, the estimated proportion of reproducible signals
- mFDR-hat(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) for each rank threshold i
- Rejection threshold N-hat = max{i : mFDR-hat(i) <= alpha} and the set of genes with M_g <= N-hat declared reproducible

## How to apply

First rank each replicate experiment (rank 1 = most abundant, ties broken by random assignment) and compute M_g = max{Rgx, Rgy} per gene (Definition 1). Estimate k-hat by minimizing the weighted mean squared error MSEn(lambda) = (n - i_lambda)^{-1} * sum_x (S_n_hat(x/n) - (1-lambda) S_lambda(x/n))^2 over i = 0, 1, ..., floor(0.9n), where S_lambda(x) = 1 - (x-lambda)^2/(1-lambda)^2 is the limiting irreproducible survival function from Theorem 1; k-hat deliberately under-estimates n*pi1 so n-k-hat over-estimates the irreproducible count. Then compute Q(i), the number of genes with M_g <= i, and mFDR-hat(i) = (i-k-hat)^2 / (Q(i)(n-k-hat)) for i = k-hat+1, ..., n, declaring all genes with M_g <= N-hat = max{i : mFDR-hat(i) <= alpha} reproducible. The estimand is pi1-hat = k-hat/n, the proportion of reproducible signals; the procedure is fully nonparametric and requires no tuning parameters.

## Related tools

- **Python (NumPy)** (Implement the MSEn(lambda) minimization for k-hat, the Q(i) cumulative counts, and the mFDR-hat(i) estimator; the closed-form MaRR estimator is implemented in NumPy/pandas for the SEQC analysis.)
- **maximum rank reproducibility (MaRR) procedure** (The method this skill implements: k-hat estimation plus mFDR-based thresholding on the maximum rank statistic.)

## Examples

```
k_hat = min((i for i in range(int(0.9*n)+1)), key=lambda i: MSEn(i/n)); mFDR = lambda i: (i-k_hat)**2/(Q(i)*(n-k_hat)); N_hat = max(i for i in range(k_hat+1, n+1) if mFDR(i) <= 0.05)
```

## Evaluation signals

- On irreproducible-only data (two independent uniform rank permutations), the empirical probability mass function of M_g/n matches (2i-1)/n^2 from Proposition 1.
- On ideal-setting simulations (e.g. pi1 = 0.35, n = 1000, assumptions I1-I3), pi1-hat converges to the true pi1 as n grows (Theorem 2).
- Across simulation settings (pi1 in {0.10, 0.30, 0.65, 0.80}, r0 in {0.30, 0.70, 0.95}), the empirical mFDR (false calls / total declared reproducible) is at or below the nominal alpha of 0.05 or 0.01.
- On the six SEQC comparisons at alpha = 0.01, pi1-hat reproduces the reported values 0.851, 0.847, 0.900, 0.898 for comparisons 1, 2, 4, 5 (vs copula-model estimates 0.773, 0.767, 0.861, 0.649).
- k-hat is at most the index i that minimizes MSEn over the truncated grid i = 0, ..., floor(0.9n), ensuring n - k-hat over-estimates the number of irreproducible genes.

## Limitations

- k-hat under-estimates n*pi1 by construction, so pi1-hat is a conservative estimate of the reproducible proportion.
- The rejection region is the square set {M_g <= N-hat} on the rank plane; in the near-perfect-correlation Simulation B setting (r0 = 0.95, pi1 = 0.80) this square boundary is broader than the copula mixture model's Gaussian-tail-shaped boundary, and conversely the copula boundary omits reproducible signals somewhat off the diagonal y = x.
- Ranks discard magnitude information, and ranking ties must be broken by random assignment before M_g is computed.

## Evidence
