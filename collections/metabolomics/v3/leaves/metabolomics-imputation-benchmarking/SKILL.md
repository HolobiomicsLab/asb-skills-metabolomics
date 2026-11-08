---
name: metabolomics-imputation-benchmarking
description: Use when targeted metabolomics (FFA/BA LC-MS) matrices contain LOD/LOQ-driven
  MNAR gaps and you must objectively choose an imputation method rather than substituting
  a single value blindly.
when_to_use_negative:
- Input is already a non-targeted GC/MS dataset where the majority of missing values
  are MCAR/MAR before manual missing retrieval — the article states GSimp's default
  left-censored settings were only evaluated for that case after missing retrieval,
  not as a general benchmarking target.
- Missing values are informative deletions from a design (e.g. QC-flagged compounds)
  rather than LOD/LOQ-driven left-censoring; MNAR simulation would not reflect the
  actual mechanism.
- You need runtime or convergence benchmarks for large-scale data — the article defers
  these to supplementary S1 Table and S2 Fig rather than the benchmarking pipeline
  itself.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: Gibbs sampler imputation method under benchmark; run via pre_processing_GS_wrapper
    with lo=-Inf, hi='min' for left-censored data
  repo: https://github.com/WandeRum/GSimp
- name: imputeLCMD
  role: provides impute.QRILC used both as GSimp's initialization and as a competing
    imputation method
- name: kNN-TN
  role: truncated-normal kNN competitor imputation method from Trunc_KNN directory
  repo: https://github.com/WandeRum/GSimp
- name: ropls
  role: performs supervised PLS-DA ordination (nPCs=2) for the multivariate Procrustes
    evaluation
- name: vegan
  role: computes Procrustes sum of squared errors between original and imputed sample
    configurations
- name: ggplot2
  role: visualizes p-value correlations and Procrustes SSE across methods and missingness
    levels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:6abb564d16a0040e3039ef95c8719544aa5ba57e468515ed781fdfeddead8f40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - PLS-DA
  - R
  - ggplot2
  - imputeLCMD
  - kNN-TN
  - ropls
  - vegan
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# metabolomics-imputation-benchmarking

## Summary

Benchmark left-censored MNAR imputation methods (GSimp, QRILC, kNN-TN, HM) on targeted metabolomics data by generating graded missingness, running per-method imputation wrappers, and scoring accuracy, distribution preservation, and downstream statistical sensitivity against the original complete data.

## When to use

Use when targeted metabolomics (FFA/BA LC-MS) matrices contain LOD/LOQ-driven MNAR gaps and you must objectively choose an imputation method rather than substituting a single value blindly.

## When NOT to use

- Input is already a non-targeted GC/MS dataset where the majority of missing values are MCAR/MAR before manual missing retrieval — the article states GSimp's default left-censored settings were only evaluated for that case after missing retrieval, not as a general benchmarking target.
- Missing values are informative deletions from a design (e.g. QC-flagged compounds) rather than LOD/LOQ-driven left-censoring; MNAR simulation would not reflect the actual mechanism.
- You need runtime or convergence benchmarks for large-scale data — the article defers these to supplementary S1 Table and S2 Fig rather than the benchmarking pipeline itself.

## Inputs

- complete scaled targeted metabolomics data matrix (e.g. real_data.csv containing FFAs and BAs, or data_sim.csv)
- group labels vector rep(c(0,1), each=80) as a factor for supervised ordination
- upstream list of MNAR-degraded/imputed datasets (sim_MNAR_list from MNAR_gen_imp)
- GSimp_evaluation.R and Impute_wrapper.R scripts from the WandeRum/GSimp repository

## Outputs

- tidy evaluation_metrics.csv with columns method, Miss_Num, t-test p-value correlation, and PLS-Procrustes SSE
- per-method imputed data matrices at each missingness level
- ggplot2 figures showing p-value correlation and Procrustes SSE across methods and missing-variable counts

## How to apply

First simulate MNAR degradation by taking the scaled complete matrix and removing values with MNAR_gen_imp at mis_var_prop=seq(.1,.8,.1) and var_mis_prop=seq(.3,.6,.1), then run each imputation wrapper (pre_processing_GS_wrapper for GSimp with iters_each=50, iters_all=10, lo=-Inf, hi='min'; impute.QRILC; kNN-TN; and HM/minimum substitution) on the degraded matrices. Evaluate imputation accuracy on the held-out entries, compare observation distributions between imputed and original values, and quantify downstream statistical sensitivity by log-transforming per-variable two-sample t-test p-values and computing Pearson correlation between imputed and original p-value vectors (Ttest_cor_cal_plot with cor='P'). For multivariate fidelity, run supervised PLS-DA ordination with nPCs=2 on group labels rep(c(0,1), each=80) via ropls and compute Procrustes sum of squared errors (vegan::procrustes) between original and imputed sample configurations. Assemble all metrics into a tidy table keyed by method, number of missing variables, and metric; the method with the highest p-value correlation and lowest PLS-Procrustes SSE (GSimp in this study) wins.

## Related tools

- **GSimp** (Gibbs sampler imputation method under benchmark; run via pre_processing_GS_wrapper with lo=-Inf, hi='min' for left-censored data) — https://github.com/WandeRum/GSimp
- **imputeLCMD** (provides impute.QRILC used both as GSimp's initialization and as a competing imputation method)
- **kNN-TN** (truncated-normal kNN competitor imputation method from Trunc_KNN directory) — https://github.com/WandeRum/GSimp
- **ropls** (performs supervised PLS-DA ordination (nPCs=2) for the multivariate Procrustes evaluation)
- **vegan** (computes Procrustes sum of squared errors between original and imputed sample configurations)
- **ggplot2** (visualizes p-value correlations and Procrustes SSE across methods and missingness levels)

## Examples

```
source('GSimp_evaluation.R'); source('GSimp.R'); sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1)); res <- pre_processing_GS_wrapper(sim_MNAR_list[[1]])
```

## Evaluation signals

- One row per method × missing proportion with both p-value correlation and PLS-Procrustes SSE populated — no missing cells in the tidy output table.
- GSimp should show the highest log-transformed t-test p-value Pearson correlation and the lowest Procrustes SSE among HM, QRILC, kNN-TN, and GSimp as missingness increases, in both FFA and BA datasets.
- MNAR degradation must be reproducible: MNAR_gen_imp called with mis_var_prop=seq(.1,.8,.1) and var_mis_prop=seq(.3,.6,.1) yields a list of degraded matrices that, when re-imputed, recover original values on non-missing entries.
- Imputed values must respect the left-censored bound: with hi='min', no imputed entry may exceed the minimum observed value of that variable (contrast kNN-TN's right-shift artifact).
- Sample-scale invariants: after scale_recover and exp() recovery, column variances of the imputed matrix should approximate those of the original scaled data before imputation.

## Limitations

- The benchmark was evaluated only on two real-world targeted datasets (FFA and BA) and one simulation dataset; extension to other -omics or complex scenarios is explicitly deferred by the authors.
- No specific version numbers were reported for glmnet, vegan, ropls, or imputeLCMD beyond the README table, and exact threshold/quantile settings for alternative truncation points (e.g. 10% quantile vs known LOQ/LOD) are not pinned down in the article.
- Runtime and convergence behavior for the parallel version on large-scale datasets are not quantified in the discussion (deferred to S1 Table and S2 Fig).

## Evidence

- [abstract] GSimp benchmarking procedure: "We compared GSimp with other three imputation methods on two real-world targeted metabolomics datasets and one simulation dataset using our imputation evaluation pipeline."
- [methods] MNAR simulation parameters: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
- [methods] GSimp core parameters: "GS_impute(., iters_each=50, iters_all=10, initial = data_raw_log_qrilc_sc_df, lo=-Inf, hi= 'min', n_cores=2, imp_model='glmnet_pred')"
- [results] evaluation metrics definition: [evidence span withheld — anchor status: not_found]
- [methods] group labels and ordination: [evidence span withheld — anchor status: not_found]
- [discussion] left-censored bound rationale: [evidence span withheld — anchor status: not_found]
- [discussion] kNN-TN artifact: "a right shift of missing part occurs, causing imputed values to exceed the truncation point"
- [discussion] HM distortion: "our results indicated that HM could severely distort the data distribution"
- [readme] pre-processing wrapper steps: "Log-transformation (for non-normal data), Initialization for missing values (e.g., QRILC), Centralization and scaling (for elastic-net prediction), Imputation using GSimp, Scaling recovery,"
- [intro] competing method provenance: "Quantile regression approach for left-censored missing (QRILC) imputes missing data using random draws from a truncated distribution"
- [readme] evaluation pipeline composition: "*GSimp_evaluation.R* contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline."
