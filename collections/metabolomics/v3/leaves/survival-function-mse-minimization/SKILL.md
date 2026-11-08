---
name: survival-function-mse-minimization
description: Use when you have per-experiment rank lists (rank 1 = most extreme) from
  two replicate high-throughput experiments and need a tuning-parameter-free estimate
  of the number of reproducible genes (k-hat/pi1-hat) without parametric assumptions
  on the dependence structure.
when_to_use_negative:
- Input is raw test statistics rather than per-experiment ranks — the survival-function
  estimator operates on ranked lists with rank 1 as the most extreme statistic
- The reproducible signals are weak and infrequent (e.g. muA=1, pi1=0.1) or form a
  large weakly correlated group (rho=0.4, pi1=0.8) — in Simulation A MaRR was anti-conservative
  in the former and underestimated pi1 in the latter, where the copula mixture model
  was favored
- A parametric dependence model is acceptable and desired — this skill exists specifically
  to avoid the copula mixture model's parametric assumptions on the dependence structure
  of reproducible genes
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Python
  role: Implementation language for the MSEn(lambda) grid search over i = 0..floor(0.9n)
    and the mFDR threshold computation (marr_core.py)
- name: NumPy
  role: Numerical computation of empirical survival functions S_n_hat(x/n) and the
    weighted theoretical survival (1-lambda)*S_lambda(x/n)
- name: R idr package
  role: Existing reproducibility method used as the comparison baseline when evaluating
    MaRR on Simulation A
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_006
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:fefe05cbbeb1d0c3ea21a3dc0e96e9ecedebdba7666955164936db8dfb6f47c9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  - R idr package
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# survival-function-mse-minimization

## Summary

Nonparametric MaRR estimator that locates the change-point from reproducible to irreproducible signals by minimizing the mean squared error between the empirical survival function of maximum rank statistics Mg = max{Rgx, Rgy} and a theoretical limiting survival function S_lambda, yielding k-hat and hence pi1-hat for two-replicate rank lists.

## When to use

Use when you have per-experiment rank lists (rank 1 = most extreme) from two replicate high-throughput experiments and need a tuning-parameter-free estimate of the number of reproducible genes (k-hat/pi1-hat) without parametric assumptions on the dependence structure.

## When NOT to use

- Input is raw test statistics rather than per-experiment ranks — the survival-function estimator operates on ranked lists with rank 1 as the most extreme statistic
- The reproducible signals are weak and infrequent (e.g. muA=1, pi1=0.1) or form a large weakly correlated group (rho=0.4, pi1=0.8) — in Simulation A MaRR was anti-conservative in the former and underestimated pi1 in the latter, where the copula mixture model was favored
- A parametric dependence model is acceptable and desired — this skill exists specifically to avoid the copula mixture model's parametric assumptions on the dependence structure of reproducible genes

## Inputs

- Two per-experiment rank lists for n genes (rank 1 = most extreme statistic), one per replicate experiment
- Maximum rank statistics Mg = max{Rgx, Rgy} per gene (derived from the rank lists)
- Nominal FDR level alpha (e.g. 0.05 or 0.01) for the downstream mFDR threshold N_hat

## Outputs

- k_hat = argmin_i MSEn(i/n) over the grid i = 0..floor(0.9n) (change-point from reproducible to irreproducible signals)
- pi1-hat (proportion of reproducible genes, n - k_hat over n)
- N_hat, the maximum rank threshold at which mFDR-hat(i) <= alpha
- Standalone implementation marr_core.py and numerical validation results (validation_report.json)

## How to apply

Convert each dataset to per-experiment ranks (rank 1 = most extreme) and compute the maximum rank statistic Mg = max{Rgx, Rgy} per Definition 1. Compute the empirical survival function S_n_hat(x/n) and the theoretical survival S_lambda(x) = 1 - (x-lambda)^2/(1-lambda)^2 for lambda <= x <= 1 from Theorem 1. Minimize MSEn(lambda) = (n - i)^{-1} * sum_x (S_n_hat(x/n) - (1-lambda) S_lambda(x/n))^2 over the grid i = 0, 1, ..., floor(0.9n), returning k_hat = argmin_i MSEn(i/n); the cap at 0.9n is a key decision point since k-hat deliberately under-estimates n*pi1 so that n - k_hat over-estimates the irreproducible count. Feed k_hat into the mFDR step, mFDR-hat(i) = (i - k_hat)^2 / (Q(i)(n - k_hat)) where Q(i) counts genes with Mg <= i, and select N_hat = max{i : mFDR-hat(i) <= alpha} at alpha = 0.05 and 0.01. Implement in Python/NumPy, e.g. as a standalone marr_core.py, and validate on irreproducible-only permutations before applying to real SEQC-style rank lists.

## Related tools

- **Python** (Implementation language for the MSEn(lambda) grid search over i = 0..floor(0.9n) and the mFDR threshold computation (marr_core.py))
- **NumPy** (Numerical computation of empirical survival functions S_n_hat(x/n) and the weighted theoretical survival (1-lambda)*S_lambda(x/n))
- **R idr package** (Existing reproducibility method used as the comparison baseline when evaluating MaRR on Simulation A)

## Examples

```
from marr_core import estimate_k_hat; k_hat = estimate_k_hat(Mg, n, grid_max=int(0.9*n))  # Mg = max(Rgx, Rgy) per gene
```

## Evaluation signals

- Empirical probability mass function of Mg/n on irreproducible-only data (two independent uniform rank permutations) matches the (2i-1)/n^2 form from Proposition 1
- k_hat under-estimates the true n*pi1 (n - k_hat over-estimates the number of irreproducible genes), so mFDR-hat remains conservative
- pi1-hat converges to the true pi1 as n grows on ideal-setting data (e.g. pi1 = 0.35, n = 1000, assumptions I1-I3), per Theorem 2
- Grid search is restricted to i <= floor(0.9n); any argmin outside this range indicates an implementation error
- Downstream empirical FDR at the N_hat threshold is at or below the nominal alpha (0.05 or 0.01) across simulation settings

## Limitations

- Anti-conservative (inflated FDR) when signals are weak and infrequent, e.g. the Simulation A setting muA=1, pi1=0.1
- Systematically underestimates pi1 for a large weakly correlated reproducible group (muA in {1,2}, rho=0.4, pi1=0.8), becoming overly conservative; the copula mixture model is favored there
- k-hat under-estimates n*pi1 by design, so pi1-hat is biased low rather than exact
- Derived for two replicates via Mg = max{Rgx, Rgy}; extending to three replicates requires re-deriving the null distribution of Mg(3) (P(Mh<=i) = i^3/n^3) as outlined in the supplementary materials

## Evidence

- [supplementary] Theoretical survival function from Theorem 1: [evidence span withheld — anchor status: not_found]
- [supplementary] Grid search definition of the estimator: [evidence span withheld — anchor status: not_found]
- [results] Rationale for the under-estimation of k-hat: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation via Proposition 1 PMF: [evidence span withheld — anchor status: not_found]
