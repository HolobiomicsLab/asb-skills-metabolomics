---
name: empirical-fdr-estimation
description: Use when you must call reproducible genes from two ranked high-throughput
  replicate lists with error control at a nominal FDR level (e.g. alpha=0.05) and
  no parametric assumptions on dependence or signal distribution.
when_to_use_negative:
- Input data are raw test statistics not yet converted to per-experiment ranks; the
  MaRR procedure operates on the rank scale via the maximum rank statistic Mg.
- 'Signals are weak and infrequent (muA=1, pi1=0.1) or there is a large weakly correlated
  reproducible group (muA in {1,2}, rho=0.4, pi1=0.8): MaRR was anti-conservative
  or systematically underestimated pi1 there, so the copula mixture model is favored
  instead.'
- The goal is discriminative-power comparison under model-based parametric assumptions
  where fitting the copula mixture model of Li et al. (2011) via EM with ten random
  initializations is the intended analysis.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Generates simulation datasets, computes per-experiment ranks, the maximum
    rank statistic Mg, and empirical FDR aggregations
- name: NumPy
  role: Draws reproducible/irreproducible signal statistics (e.g. tg,1 ~ Unif(1,5);
    tg,2 | tg,1 ~ N(tg,1, 1-rg^2)) and vectorized rank computations
- name: SciPy
  role: Survival function S-hat_n evaluations used in minimizing MSEn(i/n) for the
    k-hat change-point estimate
- name: pandas
  role: Writes per-setting, per-method empirical FDR and discriminative power results
    to CSV (e.g. simulation_b_fdr_power_results.csv)
- name: R idr package
  role: Reference implementation context for ranked-list reproducibility analysis
    against which MaRR FDR control is compared
- name: matplotlib
  role: Renders ROC curves on the rank scale for the discriminative-power assessment
    accompanying the FDR estimates
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:a13aa336d89eb8729e1baf3e5d4ac5fb1cfc521b5d1e988ec8ef8d3fa9d61e9b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  - R idr package
  - SciPy
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# empirical-fdr-estimation

## Summary

Estimates and validates empirical false discovery rates for reproducibility calls using the nonparametric MaRR maximum rank procedure, which computes marginal FDRs from the distribution of irreproducible maximum rank statistics without tuning parameters. Use it to benchmark FDR control against parametric alternatives such as the copula mixture model.

## When to use

Use when you must call reproducible genes from two ranked high-throughput replicate lists with error control at a nominal FDR level (e.g. alpha=0.05) and no parametric assumptions on dependence or signal distribution.

## When NOT to use

- Input data are raw test statistics not yet converted to per-experiment ranks; the MaRR procedure operates on the rank scale via the maximum rank statistic Mg.
- Signals are weak and infrequent (muA=1, pi1=0.1) or there is a large weakly correlated reproducible group (muA in {1,2}, rho=0.4, pi1=0.8): MaRR was anti-conservative or systematically underestimated pi1 there, so the copula mixture model is favored instead.
- The goal is discriminative-power comparison under model-based parametric assumptions where fitting the copula mixture model of Li et al. (2011) via EM with ten random initializations is the intended analysis.

## Inputs

- Two per-experiment rank lists for n genes (rank 1 = most extreme), one per replicate experiment
- Simulated or observed test statistics for reproducible signals (e.g. bivariate normal with mean (muA, muA) and covariance [[1, rho], [rho, 1]]) and irreproducible signals from N(0, I)
- Nominal error rate alpha (0.05 in the article's simulations)

## Outputs

- Change-point estimate k-hat and threshold N-hat on the maximum rank statistic
- Set of genes declared reproducible (Mg <= N-hat)
- Empirical FDR per method per setting, e.g. one row per method per setting in simulation_a_fdr_results.csv (48 rows) or simulation_b_fdr_power_results.csv

## How to apply

Convert each dataset to per-experiment ranks (rank 1 = most extreme) and compute the maximum rank statistic Mg = max(Rgx, Rgy) for each gene. Estimate the change-point k-hat by minimizing the mean squared error MSEn(i/n) = (n-i)^{-1} sum_x (S-hat_n(x/n) - (1-i/n) S_{i/n}(x))^2 over i = 0,...,floor(0.9n). Set N-hat = max{k-hat < i <= n : (i-k-hat)^2 / (Q(i)(n-k-hat)) <= 0.05} (equivalently mFDR-hat(i) <= alpha) and declare genes with Mg <= N-hat reproducible. Compute the empirical FDR as the number of false calls divided by the total number of genes declared reproducible, aggregated over repeated simulated datasets (e.g. 100 datasets per setting in the article's Simulations A and B).

## Related tools

- **Python** (Generates simulation datasets, computes per-experiment ranks, the maximum rank statistic Mg, and empirical FDR aggregations)
- **NumPy** (Draws reproducible/irreproducible signal statistics (e.g. tg,1 ~ Unif(1,5); tg,2 | tg,1 ~ N(tg,1, 1-rg^2)) and vectorized rank computations)
- **SciPy** (Survival function S-hat_n evaluations used in minimizing MSEn(i/n) for the k-hat change-point estimate)
- **pandas** (Writes per-setting, per-method empirical FDR and discriminative power results to CSV (e.g. simulation_b_fdr_power_results.csv))
- **R idr package** (Reference implementation context for ranked-list reproducibility analysis against which MaRR FDR control is compared)
- **matplotlib** (Renders ROC curves on the rank scale for the discriminative-power assessment accompanying the FDR estimates)

## Examples

```
import numpy as np; ranks = stats.rankdata(-Z, axis=0); Mg = ranks.max(axis=1); k_hat = i_grid[np.argmin([msen(ranks, i/n) for i in range(int(0.9*n))])]; N_hat = max(i for i in range(k_hat+1, n+1) if (i-k_hat)**2/(Q(i)*(n-k_hat)) <= 0.05)
```

## Evaluation signals

- MaRR's empirical FDR is approximately at or below the nominal 0.05 level across all simulation settings (e.g. the 12 Simulation B settings with pi1 in {0.10,0.30,0.65,0.80} and r0 in {0.30,0.70,0.95}).
- Known deviation reproduced: in Simulation A, MaRR's empirical FDR exceeds 0.05 in the (muA=1, pi1=0.1) weak/infrequent-signal settings.
- Known pi1 bias reproduced: MaRR's pi1-hat falls below 0.8 in the (muA in {1,2}, rho=0.4, pi1=0.8) settings, where the copula mixture model is favored.
- Results table schema: one row per method per setting (48 rows for the 24 Simulation A settings x 2 methods) with empirical FDR as false calls divided by total declared reproducible.
- FDR-based calls are consistent with the rank threshold rule: exactly the genes with Mg <= N-hat are declared reproducible.

## Limitations

- MaRR is anti-conservative (inflated FDR) when signals are weak and infrequent (muA=1, pi1=0.1).
- MaRR systematically underestimates pi1 for a large weakly correlated reproducible group (muA=1 or 2, rho=0.4, pi1=0.8), making it overly conservative in that setting and favoring the copula mixture model.
- The change-point search is restricted to i = 0,...,floor(0.9n), capping where N-hat can fall relative to n.
- The copula mixture model comparisons require EM reruns (ten per dataset) with random initializations, so fitted likelihoods and resulting FDR comparisons carry initialization variability.

## Evidence

- [abstract] abstract: "the MaRR procedure effectively controls false discovery rates, has desirable power properties, and compares well to existing methods"
- [methods] methods: [evidence span withheld — anchor status: not_found]
