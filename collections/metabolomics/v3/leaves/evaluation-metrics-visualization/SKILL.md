---
name: evaluation-metrics-visualization
description: Use when comparing left-censored (MNAR) imputation methods on targeted
  metabolomics matrices and you need a distribution-level visual metric — plot Procrustes
  SSE vs. number of missing variables to pick the least-distorting method.
when_to_use_negative:
- Data are MAR/MCAR with informative bounds not needed — use non-informative bounds
  (-Inf, +Inf) for GSimp instead of the left-censored evaluation pipeline
- Only pointwise imputation accuracy (NRMSE-type) is needed and the sample-level observation
  distribution is not of interest
- Data are post-missing-retrieval non-targeted GC/MS profiles where the MNAR assumption
  and truncation-point defaults do not apply
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0091
tools:
- name: vegan
  role: performs the 2-PC PCA ordination and computes the Procrustes sum of squared
    errors between original and imputed ordinations
- name: GSimp
  role: one of the three MNAR imputation methods being ranked; run via GS_impute (iters_each=50,
    iters_all=10, initial='qrilc', lo=-Inf, hi='min')
  repo: https://github.com/WandeRum/GSimp
- name: QRILC
  role: impute.QRILC from imputeLCMD, one of the three methods ranked on Procrustes
    SSE
  repo: https://github.com/WandeRum/GSimp
- name: kNN-TN
  role: kNN truncation imputation (imputeKNN, k=3, distance='truncation') ranked on
    the same metric
  repo: https://github.com/WandeRum/GSimp
- name: R
  role: environment hosting GSimp_evaluation.R, Impute_wrapper.R, vegan, and ggplot2
    used to build the SSE plot and table
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:78169d887fb0297846f83c2858cbdf8167ea74ead1350b13b52b724dd3b9e51a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - QRILC
  - R
  - kNN-TN
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# evaluation-metrics-visualization

## Summary

Visualize and tabulate Procrustes sum of squared errors between original and MNAR-imputed data to rank metabolomics missing-value imputation methods (GSimp, QRILC, kNN-TN) on distribution preservation.

## When to use

Use when comparing left-censored (MNAR) imputation methods on targeted metabolomics matrices and you need a distribution-level visual metric — plot Procrustes SSE vs. number of missing variables to pick the least-distorting method.

## When NOT to use

- Data are MAR/MCAR with informative bounds not needed — use non-informative bounds (-Inf, +Inf) for GSimp instead of the left-censored evaluation pipeline
- Only pointwise imputation accuracy (NRMSE-type) is needed and the sample-level observation distribution is not of interest
- Data are post-missing-retrieval non-targeted GC/MS profiles where the MNAR assumption and truncation-point defaults do not apply

## Inputs

- Complete simulated data matrix (data_sim.csv, 160 samples x 76 variables)
- Scaled version of the simulated data matrix
- Imputed matrices from GSimp, QRILC, and kNN-TN wrappers for each MNAR-degraded version
- Number of missing variables per degraded version (Miss_Num)

## Outputs

- Procrustes SSE table (pro_ss_df) with one row per method x missing-variable proportion
- Procrustes SSE versus number of missing variables plot (ggplot2, fixed colors/shapes)
- Per-method ranking of distribution preservation (lowest SSE wins)

## How to apply

Start from a complete simulated data matrix (data_sim.csv, 160 samples x 76 variables) and its scaled version, generate MNAR-degraded versions with MNAR_gen_imp using mis_var_prop=seq(.1,.8,.1) and var_mis_prop cut-offs drawn from U(0.3,0.6), then impute each with GS_impute (iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min'), QRILC (impute.QRILC), and kNN-TN (imputeKNN, k=3, distance='truncation'). For each imputed matrix, perform PCA ordination on 2 PCs with vegan (PCA_pro_list, Procrustes_cal_plot with DR='PCA', nPCs=2) and compute the Procrustes sum of squared errors between original and imputed ordinations per method and per missing-variable proportion. Plot SSE versus number of missing variables with fixed colors (#7CAE00 GSimp, #00BFC4 QRILC, #C77CFF kNN-TN) and shapes (17, 15, 3), and assemble the SSE values into a pro_ss_df table. The rationale is that the metric captures whether the imputer preserves the original sample observation distribution rather than just pointwise accuracy; validation expects GSimp to show the lowest SSE as missingness increases.

## Related tools

- **vegan** (performs the 2-PC PCA ordination and computes the Procrustes sum of squared errors between original and imputed ordinations)
- **GSimp** (one of the three MNAR imputation methods being ranked; run via GS_impute (iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min')) — https://github.com/WandeRum/GSimp
- **QRILC** (impute.QRILC from imputeLCMD, one of the three methods ranked on Procrustes SSE) — https://github.com/WandeRum/GSimp
- **kNN-TN** (kNN truncation imputation (imputeKNN, k=3, distance='truncation') ranked on the same metric) — https://github.com/WandeRum/GSimp
- **R** (environment hosting GSimp_evaluation.R, Impute_wrapper.R, vegan, and ggplot2 used to build the SSE plot and table) — https://github.com/WandeRum/GSimp

## Examples

```
source('GSimp_evaluation.R'); sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1,.8,.1)); PCA_pro_list <- Procrustes_cal_plot(DR='PCA', nPCs=2, x='Miss_Num')
```

## Evaluation signals

- pro_ss_df contains one SSE value per method (GSimp, QRILC, kNN-TN) per missing-variable proportion (10%–80%)
- GSimp's SSE curve lies below QRILC's and kNN-TN's across the missing-variable proportion range, matching the paper's finding that GSimp preserves the original observation distribution with the least distortion
- Ordination used DR='PCA' with nPCs=2 via Procrustes_cal_plot, and the SSE is the vegan procrustes sum of squared errors between original and imputed configurations
- Plot colors and shapes match the specified scheme (#7CAE00/17 for GSimp, #00BFC4/15 for QRILC, #C77CFF/3 for kNN-TN)

## Limitations

- Convergence-sensitive: small iteration counts (iters_all=10, iters_each=50) may not reach MCMC convergence, though the authors report they do not severely affect imputation accuracy on simulation data
- The Procrustes SSE ranking is demonstrated on a simulated 160x76 targeted metabolomics matrix; behavior on other -omics or very high-missingness regimes is not fully validated
- GSimp's upper truncation point (hi='min') can be too strict when minimum values are near the LOD/LOQ; quantile-based bounds (e.g., 10% quantile) are suggested but not evaluated within this visualization metric

## Evidence

- [supplementary] pro_ss_df table from Procrustes SSE across methods: "SOR on simulation dataset along with different numbers of missing variables based on four different numbers of iterations"
- [readme] GSimp default arguments for left-censored missing values: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [supplementary] imputation method comparison colors and shapes: "based on three imputation methods: QRILC (green triangle), GSimp (blue square), and kNN-TN (purple cross)"
- [methods] data_sim.csv definition: "**data_sim.csv** is the simulated dataset (details could be found in **simulation_generation.Rmd**)."
- [abstract] GSimp performance claim on distribution preservation: "The results show that GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
