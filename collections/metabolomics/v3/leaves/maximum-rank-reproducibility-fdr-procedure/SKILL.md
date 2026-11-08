---
name: maximum-rank-reproducibility-fdr-procedure
description: Use when you have ranked lists from replicate RNA-seq (or other high-throughput)
  experiments and need to identify reproducible signals with controlled false discovery
  rate, without parametric assumptions on dependence structures.
when_to_use_negative:
- When the input data are unranked raw measurements and no meaningful ranking of genes
  by effect size is possible — MaRR operates on rank lists.
- When a parametric dependence model is preferred and the copula mixture model's assumptions
  hold (e.g., when you need per-gene class posterior probabilities rather than rank-based
  FDR control).
- When the analysis goal is differential expression within a single experiment rather
  than reproducibility across replicate experiments.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implementation language for generating simulated replicate datasets and running
    the MaRR procedure
- name: NumPy
  role: Drawing reproducible test statistics (tg,1 ~ Unif(1,5), tg,2 | tg,1 ~ N(tg,1,
    1-rg^2)) and computing maximum rank statistics
- name: SciPy
  role: Statistical computations for the survival-function comparison and mFDR estimation
- name: pandas
  role: Storing per-setting, per-method empirical FDR and discriminative power results
    (simulation_b_fdr_power_results.csv)
- name: matplotlib
  role: Plotting ROC curves and FDR control comparisons corresponding to Figure 7
    behavior
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:19a72e1a338e351e1542ea7c64a1c576a012c88bbd594127545b83c90b988506
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  - SciPy
  - matplotlib
  - pandas
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# maximum-rank-reproducibility-fdr-procedure

## Summary

The MaRR procedure is a non-parametric method for assessing reproducibility of ranked gene lists from replicate high-throughput experiments. It uses a maximum rank statistic to detect the reproducible/irreproducible change point and computes marginal FDRs without parametric assumptions or tuning parameters.

## When to use

Use when you have ranked lists from replicate RNA-seq (or other high-throughput) experiments and need to identify reproducible signals with controlled false discovery rate, without parametric assumptions on dependence structures.

## When NOT to use

- When the input data are unranked raw measurements and no meaningful ranking of genes by effect size is possible — MaRR operates on rank lists.
- When a parametric dependence model is preferred and the copula mixture model's assumptions hold (e.g., when you need per-gene class posterior probabilities rather than rank-based FDR control).
- When the analysis goal is differential expression within a single experiment rather than reproducibility across replicate experiments.

## Inputs

- Ranked gene lists (or test statistics convertible to ranks) from two replicate experiments (e.g., RNA-seq technical or biological replicates, SEQC data)
- Nominal FDR level alpha (e.g., 0.05)

## Outputs

- Set of genes declared reproducible (genes with Mg <= N-hat)
- Marginal false discovery rate (mFDR) estimates per gene/rank
- Change-point estimate k-hat and reproducibility threshold N-hat
- Empirical FDR and discriminative power metrics (e.g., ROC curves on the rank scale) when evaluated on simulated data

## How to apply

Convert test statistics from each replicate into ranks (largest values most highly ranked) and compute the maximum rank statistic Mg = max{Rgx, Rgy} per gene. Estimate the change point k-hat where reproducible signals end by minimizing MSEn(i/n) over i = 0,...,floor(0.9n) between observed and theoretical survival functions. Choose N-hat = max{k-hat < i <= n : mFDR-hat(i) <= alpha} and declare genes with Mg <= N-hat reproducible, using nominal error rate alpha = 0.05. Compute marginal false discovery rates from the distribution of irreproducible maximum rank statistics; the procedure requires no tuning parameters. For benchmarking against model-based alternatives, compare discriminative power (e.g., ROC curves on the rank scale) against the copula mixture model of Li et al. (2011) in settings like r0 = 0.95, pi1 = 0.80 with linearly degrading correlation rg = (1-r0)/4 * (tg,1-1) + r0.

## Related tools

- **Python** (Implementation language for generating simulated replicate datasets and running the MaRR procedure)
- **NumPy** (Drawing reproducible test statistics (tg,1 ~ Unif(1,5), tg,2 | tg,1 ~ N(tg,1, 1-rg^2)) and computing maximum rank statistics)
- **SciPy** (Statistical computations for the survival-function comparison and mFDR estimation)
- **pandas** (Storing per-setting, per-method empirical FDR and discriminative power results (simulation_b_fdr_power_results.csv))
- **matplotlib** (Plotting ROC curves and FDR control comparisons corresponding to Figure 7 behavior)

## Examples

```
import numpy as np; Mg = np.maximum(rank_x, rank_y); k_hat = np.argmin([mse(i/n) for i in range(int(0.9*n))]); N_hat = max(i for i in range(k_hat+1, n+1) if mfdr_hat(i) <= 0.05)
```

## Evaluation signals

- MaRR must effectively control empirical FDR at or below the nominal 0.05 level across all 12 settings examined (pi1 in {0.10,0.30,0.65,0.80}, r0 in {0.30,0.70,0.95})
- MaRR must show higher discriminative power than the copula mixture model when r0 = 0.95 and pi1 = 0.80 (Simulation B, RNA-seq-like settings)
- The copula mixture model must be more conservative than MaRR when r0 = 0.95, matching reported Simulation B behavior in Figure 7
- k-hat estimation must succeed via MSE minimization: MSEn(i/n) minimized over i = 0,...,floor(0.9n) should localize the change from reproducible to irreproducible signals
- Per-setting, per-method results should be complete in simulation_b_fdr_power_results.csv (12 settings x 2 methods x empirical FDR and power)

## Limitations

- Non-parametric by design: does not provide parametric dependence structure estimates or per-gene class assignments like the copula mixture model
- Performance depends on correlation structure between replicates; in Simulation B, correlation degrades linearly with tg,1, and MaRR's power advantage is specific to settings like r0 = 0.95, pi1 = 0.80
- The change-point search is restricted to i = 0,...,floor(0.9n), which may miss change points beyond 90% of ranked genes

## Evidence

- [results] Simulation B finding: [evidence span withheld — anchor status: not_found]
