---
name: bivariate-normal-mixture-data-simulation
description: 'Use when benchmarking reproducibility methods on rank lists: you need
  synthetic replicate-experiment data with known proportions (pi1) and correlation
  (rho) of reproducible signals to compute empirical FDR under both favorable and
  unfavorable model assumptions.'
when_to_use_negative:
- The goal is to assess reproducibility of real experimental rank lists (e.g. SEQC
  RNA-seq data) rather than to benchmark method properties on data with known generating
  parameters.
- The evaluated method relies on the copula mixture model's own parametric assumptions
  and you need only its favorable-case behavior — note that Simulation A, where data
  follow the copula mixture model's assumptions, is precisely the setting where MaRR
  can be anti-conservative or overly conservative.
- Signals cannot be reasonably modeled as bivariate normal (reproducible component
  with covariance [[1, rho], [rho, 1]], irreproducible standard normal); other generating
  distributions require a different simulation design.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Generates the bivariate normal mixture datasets, converts them to per-experiment
    ranks, runs the MaRR procedure, and computes empirical FDR per method per setting.
- name: R idr package
  role: Fits the copula mixture model of Li et al. (2011) via EM to the same ranked
    simulation data for comparison against MaRR.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/01621459.2017.1397521
    title: 'Maximum Rank Reproducibility: A Nonparametric Approach to Assessing Reproducibility
      in Replicate Experiments'
  source_content_hash: sha256:62e198f99e4661065e7fb5430a82b6cacd3bbca0eff17222fe16bced17cd9acd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - R idr package
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# bivariate-normal-mixture-data-simulation

## Summary

Simulates rank-list reproducibility benchmarks by generating replicate-experiment test statistics from bivariate normal mixtures of reproducible and irreproducible signals, enabling false discovery rate control comparisons between MaRR and model-based methods like the copula mixture model.

## When to use

Use when benchmarking reproducibility methods on rank lists: you need synthetic replicate-experiment data with known proportions (pi1) and correlation (rho) of reproducible signals to compute empirical FDR under both favorable and unfavorable model assumptions.

## When NOT to use

- The goal is to assess reproducibility of real experimental rank lists (e.g. SEQC RNA-seq data) rather than to benchmark method properties on data with known generating parameters.
- The evaluated method relies on the copula mixture model's own parametric assumptions and you need only its favorable-case behavior — note that Simulation A, where data follow the copula mixture model's assumptions, is precisely the setting where MaRR can be anti-conservative or overly conservative.
- Signals cannot be reasonably modeled as bivariate normal (reproducible component with covariance [[1, rho], [rho, 1]], irreproducible standard normal); other generating distributions require a different simulation design.

## Inputs

- Simulation setting grid: signal strength muA, signal correlation rho, and signal proportion pi1 (e.g. 24 settings across muA in {1,2}, rho in {0, 0.4}, pi1 in {0.1, 0.8})
- Per-setting random seed or generator specification for 100 replicate datasets of n=2000 genes
- Rank-conversion rule (rank 1 = most extreme test statistic) shared across evaluated methods

## Outputs

- 100 synthetic bivariate-normal-mixture datasets per setting (n=2000 genes, reproducible plus irreproducible signal components)
- Per-experiment rank lists derived from each simulated dataset
- Empirical FDR results table, one row per method per setting (48 rows for Simulation A), e.g. simulation_a_fdr_results.csv
- Estimated pi1-hat from MaRR per dataset for checking proportion-recovery bias

## How to apply

For each simulation setting, generate 100 datasets of n=2000 genes in which reproducible signals are drawn from a bivariate normal with mean (muA, muA) and covariance [[1, rho], [rho, 1]], and irreproducible signals from a standard bivariate normal, treating large test statistics as highly ranked. Convert each dataset to per-experiment ranks (rank 1 = most extreme) so downstream methods like MaRR and the copula mixture model of Li et al. can consume them identically. Sweep the generating parameters across 24 settings (e.g. muA in {1,2}, rho in {0, 0.4}, pi1 in {0.1, 0.8}) to probe both anti-conservative and overly conservative behavior. Fit the copula mixture model with an EM algorithm run ten times per dataset, initializing muA, rho, and pi1 from uniform draws over muA +/- 0.50, rho +/- 0.10, pi1 +/- 0.10 with sigma fixed at 1, retaining the highest-likelihood fit. Compute the empirical false discovery rate as false calls divided by genes declared reproducible, and verify the simulation recovers known patterns: MaRR inflates FDR above the nominal 0.05 when muA=1, pi1=0.1, and underestimates pi1 for the large weakly correlated group (rho=0.4, pi1=0.8).

## Related tools

- **Python** (Generates the bivariate normal mixture datasets, converts them to per-experiment ranks, runs the MaRR procedure, and computes empirical FDR per method per setting.)
- **R idr package** (Fits the copula mixture model of Li et al. (2011) via EM to the same ranked simulation data for comparison against MaRR.)

## Examples

```
X = rng.multivariate_normal([muA, muA], [[1, rho], [rho, 1]], size=int(pi1*n)); X0 = rng.multivariate_normal([0, 0], np.eye(2), size=n - X.shape[0]); ranks = np.argsort(np.vstack([X, X0]), axis=0)
```

## Evaluation signals

- In the (muA=1, pi1=0.1) settings, the empirical FDR of MaRR exceeds the nominal 0.05 level (anti-conservative behavior), matching Simulation A patterns.
- In the (muA in {1,2}, rho=0.4, pi1=0.8) settings, MaRR's pi1-hat falls below 0.8, indicating systematic underestimation of the large weakly correlated reproducible group.
- The copula mixture model is favored over MaRR in the large weakly correlated group settings, reproducing the reported Simulation A comparison.
- Output table has exactly one row per method per setting (48 rows for Simulation A) with empirical FDR = false calls / genes declared reproducible.
- Each dataset uses n=2000 genes and 100 replicates per setting, with irreproducible signals generated from the standard bivariate normal and reproducible signals from the correlated bivariate normal.

## Limitations

- When data are generated from the copula mixture model's own parametric assumptions (Simulation A), MaRR is anti-conservative (inflated FDR) for weak, infrequent signals (muA=1, pi1=0.1).
- MaRR systematically underestimates pi1 for a large weakly correlated reproducible group (muA=1 or 2, rho=0.4, pi1=0.8), making it overly conservative in that setting and favoring the copula mixture model.
- The simulation restricts irreproducible signals to the standard bivariate normal and reproducible signals to unit-variance correlated normals; results may not generalize to other dependence structures.

## Evidence

- [methods] Simulation A generator: reproducible/irreproducible mixture components and parameters: [evidence span withheld — anchor status: not_found]
- [results] FDR anti-conservatism for weak infrequent signals: [evidence span withheld — anchor status: not_found]
- [results] pi1 underestimation for large weakly correlated group: [evidence span withheld — anchor status: not_found]
- [methods] Copula mixture model EM fitting protocol: [evidence span withheld — anchor status: not_found]
- [methods] Nominal FDR level used in MaRR procedure: [evidence span withheld — anchor status: not_found]
