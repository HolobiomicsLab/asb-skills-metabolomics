---
name: monte-carlo-rank-data-simulation
description: Use when validating or benchmarking reproducibility procedures (MaRR,
  copula mixture model) on synthetic replicate data where ground truth pi1 and r0
  are known and empirical FDR/power must be measured.
when_to_use_negative:
- The goal is calling reproducible genes on an observed rank list — use the MaRR k-hat
  / mFDR-hat estimation procedure on real data instead of simulation.
- A parametric dependence model for replicate test statistics is desired (e.g. the
  copula mixture model of Li et al. 2011) rather than nonparametric rank-based simulation.
- Only a single experiment (no replicate) is available — the simulated structure is
  bivariate (two rank permutations or two correlated test statistics per gene).
tools:
- name: Python
  role: Language in which the irreproducible-only permutations and Simulation B data-generating
    models are implemented
- name: NumPy
  role: Draws independent uniform rank permutations and bivariate normal signals N(0,
    I) and computes M_g = max{Rgx, Rgy}
- name: SciPy
  role: Supports the normal/uniform sampling and ROC computation used in the Simulation
    B comparison
- name: pandas
  role: Writes per-setting, per-method empirical FDR and discriminative power to simulation_b_fdr_power_results.csv
- name: matplotlib
  role: Plots discriminative power (ROC curves) for MaRR vs the copula mixture model
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:b0ef9d7d09b82f20c041831b4ac2c0ee43efe6c4835e94f5010fb64e7dfa9919
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  - SciPy
  - matplotlib
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# monte-carlo-rank-data-simulation

## Summary

Generates synthetic two-experiment rank/statistic datasets with known reproducible and irreproducible gene fractions to validate the MaRR maximum-rank procedure, including irreproducible-only null data whose maximum rank statistic M_g = max{Rgx, Rgy} follows the (2i-1)/n^2 law, and Simulation B data mimicking RNA-seq with correlation degrading linearly with t_g,1.

## When to use

Use when validating or benchmarking reproducibility procedures (MaRR, copula mixture model) on synthetic replicate data where ground truth pi1 and r0 are known and empirical FDR/power must be measured.

## When NOT to use

- The goal is calling reproducible genes on an observed rank list — use the MaRR k-hat / mFDR-hat estimation procedure on real data instead of simulation.
- A parametric dependence model for replicate test statistics is desired (e.g. the copula mixture model of Li et al. 2011) rather than nonparametric rank-based simulation.
- Only a single experiment (no replicate) is available — the simulated structure is bivariate (two rank permutations or two correlated test statistics per gene).

## Inputs

- Simulation design parameters: number of genes n (e.g. 1,000 or 10,000), true reproducible proportion pi1, null correlation floor r0 (Simulation B grid: pi1 in {0.10,0.30,0.65,0.80}, r0 in {0.30,0.70,0.95})
- Number of Monte Carlo replicates per setting (100 datasets in Simulation B)
- Random seed / RNG state for reproducible permutations and normal draws (Python/NumPy)

## Outputs

- Simulated rank matrices / maximum rank statistics M_g = max{Rgx, Rgy} per gene for each replicate dataset
- Per-setting, per-method empirical FDR and discriminative power (e.g. ROC curves on the rank scale), aggregated over 100 datasets, saved as simulation_b_fdr_power_results.csv
- Numerical validation results for null PMF (2i-1)/n^2 and pi1-hat convergence, saved as validation_report.json

## How to apply

Draw reproducible test statistics with t_g,1 ~ Unif(1,5) and t_g,2 | t_g,1 ~ N(t_g,1, 1-r_g^2), where r_g = (1-r0)/4 * (t_g,1-1) + r0 so correlation degrades linearly from perfect at t_g,1=5 to r0 at t_g,1=1; draw irreproducible signals from the standard bivariate normal N(0, I) per equation (12). For null-only validation, instead draw two independent uniform rank permutations for n genes and compute M_g = max{Rgx, Rgy} per Definition 1. Convert test statistics to ranks (largest values most highly ranked) before feeding downstream procedures. For Simulation B, generate 100 datasets of n=10,000 genes per each of the 12 settings (pi1 in {0.10,0.30,0.65,0.80}, r0 in {0.30,0.70,0.95}) and aggregate per-setting, per-method empirical FDR and discriminative power. For ideal-setting validation of Theorem 2, use e.g. pi1 = 0.35, n = 1000 under assumptions (I1)-(I3) and confirm pi1-hat converges to pi1 as n grows.

## Related tools

- **Python** (Language in which the irreproducible-only permutations and Simulation B data-generating models are implemented)
- **NumPy** (Draws independent uniform rank permutations and bivariate normal signals N(0, I) and computes M_g = max{Rgx, Rgy})
- **SciPy** (Supports the normal/uniform sampling and ROC computation used in the Simulation B comparison)
- **pandas** (Writes per-setting, per-method empirical FDR and discriminative power to simulation_b_fdr_power_results.csv)
- **matplotlib** (Plots discriminative power (ROC curves) for MaRR vs the copula mixture model)

## Examples

```
import numpy as np; R = np.stack([np.random.permutation(n) + 1 for _ in range(2)]); M = R.max(axis=0)  # irreproducible-only max rank statistic
```

## Evaluation signals

- Empirical probability mass function of M_g/n on irreproducible-only data matches (2i-1)/n^2 from Proposition 1.
- Ideal-setting data (e.g. pi1 = 0.35, n = 1000) under assumptions (I1)-(I3) show pi1-hat converging to pi1 as n grows (Theorem 2).
- In Simulation B, empirical FDR for MaRR is approximately at or below the nominal 0.05 level across all 12 settings.
- At r0 = 0.95 and pi1 = 0.80, MaRR shows higher discriminative power than the copula mixture model, and the copula model is more conservative than MaRR, matching the reported Simulation B behavior in Figure 7.
- Reproducible-gene correlation matches the design: r_g = (1-r0)/4 * (t_g,1-1) + r0, i.e. perfect correlation at t_g,1 = 5 degrading to r0 at t_g,1 = 1.

## Limitations

- The generative model is specific: Simulation B assumes t_g,1 ~ Unif(1,5) with linearly degrading correlation and N(0, I) irreproducible signals; datasets from other dependence structures may not satisfy the limiting survival function S_lambda(x) = 1 - (x-lambda)^2/(1-lambda)^2 used by MaRR.
- The setup is bivariate (two replicate experiments); extension to more than two replicates is not covered by this simulation design.
- Simulation B is expensive: 100 datasets of n = 10,000 genes across 12 settings, and the copula mixture model comparison requires 10 restarts per dataset with random initial parameters.

## Evidence

- [methods] null simulation design: [evidence span withheld — anchor status: not_found]
- [methods] Simulation B generative model: [evidence span withheld — anchor status: not_found]
- [methods] null PMF check: [evidence span withheld — anchor status: not_found]
- [methods] Simulation B purpose: [evidence span withheld — anchor status: not_found]
