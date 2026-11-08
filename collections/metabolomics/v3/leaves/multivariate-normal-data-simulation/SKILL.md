---
name: multivariate-normal-data-simulation
description: Use when benchmarking missing-value imputation methods on high-dimensional
  metabolomics-like matrices and you need replicated simulated data with known true
  values, specified correlation structure, and controlled MNAR/MAR missingness for
  RMSE evaluation.
when_to_use_negative:
- Input is real observed metabolomics data where the goal is imputation on actual
  measurements rather than simulation with known ground truth
- The analysis goal is downstream statistical-testing concordance (e.g., MLCI on real
  datasets) rather than RMSE-based simulation benchmarking
- You need realistic non-normal metabolite distributions — this simulation relies
  on the multivariate normal assumption, which the paper itself flags as a limitation
  of KNN-TN
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3174
tools:
- name: R
  role: Runtime-implemented simulation and imputation scripts used to generate the
    multivariate normal replicate datasets, inject missingness, and run KNN-TN/KNN-CR/KNN-EU
    imputation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:8f9e50bd2b3c8a689263130cde93d891de539f0aaa409b1c5f8d84f11dc6bb60
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R (runtime-implemented simulation and imputation scripts)
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# multivariate-normal-data-simulation

## Summary

Generates synthetic high-dimensional metabolomics-like datasets by drawing samples from an M-dimensional multivariate normal with controlled correlation structures, so that imputation methods (KNN-TN, KNN-CR, KNN-EU) can be benchmarked against known ground-truth values under controlled MNAR/MAR missingness.

## When to use

Use when benchmarking missing-value imputation methods on high-dimensional metabolomics-like matrices and you need replicated simulated data with known true values, specified correlation structure, and controlled MNAR/MAR missingness for RMSE evaluation.

## When NOT to use

- Input is real observed metabolomics data where the goal is imputation on actual measurements rather than simulation with known ground truth
- The analysis goal is downstream statistical-testing concordance (e.g., MLCI on real datasets) rather than RMSE-based simulation benchmarking
- You need realistic non-normal metabolite distributions — this simulation relies on the multivariate normal assumption, which the paper itself flags as a limitation of KNN-TN

## Inputs

- Simulation configuration: dimensions (20x400, 50x400, 100x900), number of replications (100), metabolite means drawn from Uniform(-5, 5)
- Correlation structure specification: blockwise positive (rho_w = 0.7, rho_off = 0.2), AR(1) (rho = 0.9), or blockwise mixed (positive within, negative between blocks)
- Missingness design: percent missing (9%, 15%, 30%), 2:1 MNAR-to-MAR split, and LOD quantile defining MNAR values

## Outputs

- 100 replicate complete (ground-truth) data matrices per size/correlation configuration
- Masked versions of each matrix with injected 2:1 MNAR-to-MAR missingness at 9%, 15%, and 30%
- Ground-truth-to-masked pairs enabling per-replication, per-method RMSE computation and ANOVA contrasts

## How to apply

Generate replicate datasets by drawing samples from an M-dimensional multivariate normal whose metabolite means are drawn from a Uniform(-5, 5) distribution, for each of three correlation structures: blockwise positive (within-block rho_w = 0.7, between-block rho_off = 0.2), AR(1) with rho = 0.9, and blockwise mixed (positive within, negative between blocks), at dimensions 20x400, 50x400, and 100x900, with 100 replications per configuration. After simulating each complete matrix, introduce 9%, 15%, and 30% missingness with a 2:1 MNAR-to-MAR split — MNAR values defined as data below the corresponding quantile LOD, MAR values removed at random — and retain the complete matrix as ground truth for subsequent RMSE computation. Apply the cleaning step that removes metabolites with more than 75% missing observations before imputation, mirroring the paper's preprocessing. In the source work this simulation was implemented in R as runtime-implemented simulation and imputation scripts feeding KNN imputation with K = 10 neighbors, and the resulting RMSEs were analyzed with a multi-factor ANOVA (method, percent missing, correlation type).

## Related tools

- **R** (Runtime-implemented simulation and imputation scripts used to generate the multivariate normal replicate datasets, inject missingness, and run KNN-TN/KNN-CR/KNN-EU imputation)

## Examples

```
library(MASS); Sigma <- make_cor_struct(type = "blockwise", rho_w = 0.7, rho_off = 0.2); X <- mvrnorm(n = 50, mu = runif(400, -5, 5), Sigma = Sigma)
```

## Evaluation signals

- Each configuration yields exactly 100 replicate datasets per dimension/correlation combination (20x400, 50x400, 100x900)
- Simulated matrix means fall within the Uniform(-5, 5) support and empirical covariance matches the specified structure (rho_w = 0.7 / rho_off = 0.2, AR(1) rho = 0.9, or blockwise mixed)
- Injected missingness per matrix matches the target 9%, 15%, or 30% with an approximately 2:1 MNAR-to-MAR split
- MNAR mask positions coincide with values below the corresponding quantile LOD; MAR positions are random
- Metabolites with more than 75% missing observations are removed by the cleaning step before imputation, and complete ground-truth matrices are retained for RMSE computation

## Limitations

- The multivariate normal assumption is a simplification; the paper notes the reliance on normality for estimating the truncated mean and standard deviation while metabolite data are often non-normal
- Estimation related to truncated distributions is unstable with excessive missing data or small samples, giving higher RMSE variability (relevant when simulating small-n, high-missingness configurations)
- Results for sample sizes under 20 and missing percentages below those evaluated (9%, 15%, 30%) were referenced but not shown ('data not shown'), so the simulation grid should not be extrapolated outside the evaluated settings

## Evidence

- [methods] workflow_generation: [evidence span withheld — anchor status: not_found]
- [methods] workflow_correlation: [evidence span withheld — anchor status: not_found]
- [methods] workflow_missingness: [evidence span withheld — anchor status: not_found]
- [discussion] article_normality: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
