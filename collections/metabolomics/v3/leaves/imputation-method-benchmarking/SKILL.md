---
name: imputation-method-benchmarking
description: Use when selecting a missing value imputation method for MNAR left-censored
  metabolomics (LOD/LOQ-caused) data, or when building an evaluation pipeline that
  must rank candidate imputers by TPR, RMSE, or SOR metrics.
when_to_use_negative:
- When missing values are predominantly MCAR/MAR (e.g. post-missing-retrieval non-targeted
  GC/MS data), since the benchmark's MNAR-generation and left-censored truncation
  bounds (lo=-Inf, hi='min') are designed for MNAR left-censored situations.
- When the goal is simply to impute a single dataset with an already-validated method,
  rather than to compare or rank multiple imputation methods.
- When the data are non-omics without left-censored (LOD/LOQ) structure, since GSimp's
  truncation-bounded Gibbs sampler and the MNAR benchmark design are specific to that
  setting.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: GSimp
  role: 'Candidate imputation method under benchmark: Gibbs sampler based left-censored
    imputation executed via GS_impute or pre_processing_GS_wrapper with glmnet prediction
    model'
  repo: https://github.com/WandeRum/GSimp
- name: QRILC
  role: Competing imputation method (impute.QRILC from imputeLCMD) applied on log-transformed
    data, also used as initialization for GSimp
  repo: https://github.com/WandeRum/GSimp
- name: kNN-TN
  role: Competing imputation method (Trunc_KNN imputeKNN with k=3, distance='truncation')
    for left-censored MNAR missing values
  repo: https://github.com/WandeRum/GSimp
- name: glmnet
  role: Prediction model (elastic net) embedded in the GS_impute Gibbs sampler during
    benchmarking
  repo: https://github.com/WandeRum/GSimp
- name: imputeLCMD
  role: R package supplying impute.QRILC used both as a competing method and as initialization
    in pre_processing_GS_wrapper
  repo: https://github.com/WandeRum/GSimp
- name: MNAR_gen_imp
  role: 'Benchmark data generator: creates MNAR-degraded matrices with mis_var_prop=seq(.1,.8,.1)
    and var_mis_prop=seq(.3,.6,.1) from scaled simulation data'
  repo: https://github.com/WandeRum/GSimp
- name: Ttest_P_cal_plot
  role: Evaluation function computing TPR = |S ∩ S'| / |S| for t-test significant
    variables at p_cut=0.05/0.01 against Miss_Num
  repo: https://github.com/WandeRum/GSimp
- name: MVI-evaluation
  role: Evaluation pipeline repository containing missing generation, imputations,
    and evaluation functions (Imputation evaluations.R)
  repo: https://github.com/WandeRum/MVI-evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:78e80cf7b81c44a5b76cecad2a7ddb87b4ec6612766bfc7ac090f4a221cdc4b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - MNAR_gen_imp
  - MVI-evaluation
  - QRILC
  - R
  - Ttest_P_cal_plot
  - glmnet
  - imputeLCMD
  - kNN-TN
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 27
---

# imputation-method-benchmarking

## Summary

A controlled benchmarking protocol that compares left-censored missing value imputation methods (GSimp, QRILC, kNN-TN, HM) on real-world targeted metabolomics data and an MNAR-degraded simulation, across imputation accuracy, distribution preservation, and downstream statistical sensitivity.

## When to use

Use when selecting a missing value imputation method for MNAR left-censored metabolomics (LOD/LOQ-caused) data, or when building an evaluation pipeline that must rank candidate imputers by TPR, RMSE, or SOR metrics.

## When NOT to use

- When missing values are predominantly MCAR/MAR (e.g. post-missing-retrieval non-targeted GC/MS data), since the benchmark's MNAR-generation and left-censored truncation bounds (lo=-Inf, hi='min') are designed for MNAR left-censored situations.
- When the goal is simply to impute a single dataset with an already-validated method, rather than to compare or rank multiple imputation methods.
- When the data are non-omics without left-censored (LOD/LOQ) structure, since GSimp's truncation-bounded Gibbs sampler and the MNAR benchmark design are specific to that setting.

## Inputs

- data_sim.csv — simulated targeted metabolomics dataset from simulation_generation.Rmd, scaled before MNAR degradation
- real_data.csv — real-world targeted metabolomics dataset containing FFAs and BAs
- targeted_data.csv — targeted LC/MS data with missing values
- untargeted_data.csv — untargeted GC/MS data with missing values after manual missing retrieval
- MNAR-degraded data matrices produced by MNAR_gen_imp

## Outputs

- Imputed data matrices from each method wrapper (GSimp, QRILC, kNN-TN, HM)
- TPR results table (e.g. tpr_results.csv) keyed by imputation method, missing proportion, and p-cutoff
- TPR plots of statistical sensitivity vs Miss_Num at p_cut=0.05 and p_cut=0.01
- Imputation accuracy metrics (RMSE/MAE) and distribution preservation / SOR plots per method

## How to apply

Generate MNAR-degraded datasets with MNAR_gen_imp on the scaled simulation matrix (data_sim.csv, 80 observations per group) using mis_var_prop=seq(.1, .8, .1) and var_mis_prop=seq(.3, .6, .1), so the MNAR cut-off is drawn from U(0.3, 0.6). Wrap each competing imputer in a *-wrapper function registered in Impute_wrapper.R: QRILC (impute.QRILC on log-transformed data), GSimp via pre_processing_GS_wrapper or GS_impute with iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred', and kNN-TN via imputeKNN with k=3, distance='truncation', perc=0. Evaluate accuracy by comparing imputed values to known true values on MNAR positions, distribution preservation via quantile-quantile or observation distribution plots, and statistical sensitivity by computing TPR = |S intersect S'| / |S| with Ttest_P_cal_plot, where S is the set of t-test significant variables (p_cut=0.05 and 0.01) in the complete original data and S' the set in the imputed data, plotted against Miss_Num. The decision rule from the paper is that GSimp achieves the highest overall TPR across different numbers of missing variables at both p-cutoffs, indicating it impairs sensitivity the least among QRILC, kNN-TN, and GSimp.

## Related tools

- **GSimp** (Candidate imputation method under benchmark: Gibbs sampler based left-censored imputation executed via GS_impute or pre_processing_GS_wrapper with glmnet prediction model) — https://github.com/WandeRum/GSimp
- **QRILC** (Competing imputation method (impute.QRILC from imputeLCMD) applied on log-transformed data, also used as initialization for GSimp) — https://github.com/WandeRum/GSimp
- **kNN-TN** (Competing imputation method (Trunc_KNN imputeKNN with k=3, distance='truncation') for left-censored MNAR missing values) — https://github.com/WandeRum/GSimp
- **glmnet** (Prediction model (elastic net) embedded in the GS_impute Gibbs sampler during benchmarking) — https://github.com/WandeRum/GSimp
- **imputeLCMD** (R package supplying impute.QRILC used both as a competing method and as initialization in pre_processing_GS_wrapper) — https://github.com/WandeRum/GSimp
- **MNAR_gen_imp** (Benchmark data generator: creates MNAR-degraded matrices with mis_var_prop=seq(.1,.8,.1) and var_mis_prop=seq(.3,.6,.1) from scaled simulation data) — https://github.com/WandeRum/GSimp
- **Ttest_P_cal_plot** (Evaluation function computing TPR = |S ∩ S'| / |S| for t-test significant variables at p_cut=0.05/0.01 against Miss_Num) — https://github.com/WandeRum/GSimp
- **MVI-evaluation** (Evaluation pipeline repository containing missing generation, imputations, and evaluation functions (Imputation evaluations.R)) — https://github.com/WandeRum/MVI-evaluation

## Examples

```
source('GSimp_evaluation.R'); source('Impute_wrapper.R'); sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1))
```

## Evaluation signals

- TPR (sensitivity for detecting differential variables) computed as |S ∩ S'| / |S| should show GSimp attaining the overall highest TPR across different Miss_Num values at both p_cut=0.05 and p_cut=0.01, exceeding QRILC and kNN-TN.
- TPR table must be keyed by imputation method × missing proportion × p-cutoff; verify Mis_Num sweep spans mis_var_prop=seq(.1,.8,.1) with MNAR cut-offs drawn from U(0.3,0.6).
- Distribution preservation check: imputed value distribution should not show the severe distortion seen with HM, nor the right-shift beyond the truncation point seen with kNN-TN; QRILC should preserve overall distribution and variances.
- GSimp imputed values must remain within the truncation bound hi='min' (or the quantile-based upper bound) and follow a favorable linear trend between imputed and original values.
- Convergence sanity: iters_each=50, iters_all=10 should not severely degrade accuracy relative to iters_all=20, iters_each=100, as tested on simulation data.

## Limitations

- Parameter values for QRILC and kNN-TN are not fully specified in the paper discussion beyond 'parameters of missing variables estimated with truncated normal distributions'; version numbers of R packages are not pinned in the discussion.
- Simulation covariance was taken from a diabetes dataset (P=76) with sample means from N(0, 0.5^2); the authors note other complex scenarios (other -omics) still need evaluation in the future.
- For non-targeted GC/MS data the majority of missing values were assumed MCAR/MAR before manual missing retrieval; GSimp's default settings are described as applicable to post-missing-retrieval data rather than raw un-retrieved spectra.
- Quantile-based upper bounds (e.g. 10% quantile via sapply(data, function(x) quantile(x, .1, na.rm=T))) are offered when hi='min' is too strict, but the paper does not prescribe which quantile suits every condition.
- Runtime/convergence behavior depends on iteration counts; large iterations (iters_all=20, iters_each=100) are recommended for convergence but cost more compute.

## Evidence

- [abstract] GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity.: "GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
- [methods] MNAR generation parameters: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
- [readme] GSimp wrapper parameters: "result <- data_raw_log_sc %>% GS_impute(., iters_each=50, iters_all=10, 
                                             initial = data_raw_log_qrilc_sc_df,"
- [supplementary] Competing imputers listed together: "based on three imputation methods: QRILC (green triangle), GSimp (blue square), and kNN-TN (purple cross)"
- [results] Source card finding on TPR: [evidence span withheld — anchor status: not_found]
- [discussion] HM vs QRILC distribution behavior: "QRILC kept the overall data distribution and variances"
- [discussion] kNN-TN right-shift failure mode: "a right shift of missing part occurs, causing imputed values to exceed the truncation point"
- [readme] Evaluation pipeline location: [evidence span withheld — anchor status: external_doc]
