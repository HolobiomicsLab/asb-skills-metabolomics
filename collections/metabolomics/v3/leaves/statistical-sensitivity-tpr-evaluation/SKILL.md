---
name: statistical-sensitivity-tpr-evaluation
description: Use when comparing imputation methods (e.g., GSimp vs QRILC vs kNN-TN)
  on left-censored/MNAR metabolomics data by measuring degradation of t-test statistical
  sensitivity, not just imputation accuracy.
when_to_use_negative:
- The missing values are MCAR/MAR (e.g., the non-targeted GC/MS data before missing
  retrieval) rather than left-censored MNAR from LOD/LOQ
- The goal is imputation accuracy or distribution preservation only, without assessing
  downstream univariate statistical sensitivity
- The data lack a complete (unimputed) reference against which the t-test significant
  variable set S can be defined
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3315
tools:
- name: GSimp (GS_impute)
  role: Gibbs-sampler imputation method under evaluation; wrapper sim_GS_wrapper with
    iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred'
  repo: https://github.com/WandeRum/GSimp
- name: QRILC (impute.QRILC, imputeLCMD)
  role: Comparison imputation method; wrapper sim_QRILC_wrapper applied on log-transformed
    data
- name: kNN-TN (Trunc_KNN)
  role: Comparison imputation method; wrapper sim_trKNN_wrapper with imputeKNN k=3,
    distance='truncation', perc=0 on log-transformed data
  repo: https://github.com/WandeRum/GSimp
- name: glmnet
  role: Elastic-net prediction model used inside GS_impute during imputation of each
    degraded dataset
- name: Ttest_P_cal_plot
  role: Evaluation function computing TPR = |S ∩ S'| / |S| at p_cut=0.05 and p_cut=0.01
    and producing TPR-vs-Miss_Num plots
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:64ef54b1b1e68f6dcf112155cf8173e1ab9b9aab140ec0595e856bfa8e11cd01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - GSimp (GS_impute)
  - QRILC
  - QRILC (impute.QRILC, imputeLCMD)
  - R
  - Ttest_P_cal_plot
  - glmnet
  - imputeLCMD
  - kNN-TN
  - kNN-TN (Trunc_KNN)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# statistical-sensitivity-tpr-evaluation

## Summary

Evaluates how MNAR-degraded, imputed metabolomics datasets preserve the ability to detect true differential variables, by computing True Positive Rate (TPR = |S ∩ S'| / |S|) against the original complete data across increasing missing-variable numbers and comparing imputation methods.

## When to use

Use when comparing imputation methods (e.g., GSimp vs QRILC vs kNN-TN) on left-censored/MNAR metabolomics data by measuring degradation of t-test statistical sensitivity, not just imputation accuracy.

## When NOT to use

- The missing values are MCAR/MAR (e.g., the non-targeted GC/MS data before missing retrieval) rather than left-censored MNAR from LOD/LOQ
- The goal is imputation accuracy or distribution preservation only, without assessing downstream univariate statistical sensitivity
- The data lack a complete (unimputed) reference against which the t-test significant variable set S can be defined

## Inputs

- Scaled simulation dataset data_sim.csv with a two-group factor (80 observations per group)
- GSimp_evaluation.R (MNAR generation and evaluation functions) and Impute_wrapper.R source scripts
- Original complete (unimputed) data matrix defining the reference significant-variable set S

## Outputs

- TPR table tpr_results.csv keyed by imputation method, missing proportion, and p-cutoff (0.05 and 0.01)
- Two TPR-vs-Miss_Num plots comparing QRILC, GSimp, and kNN-TN

## How to apply

Scale the simulated data (data_sim.csv) and define a two-group factor with 80 observations per group as in the simulation design. Generate a series of MNAR-degraded datasets with MNAR_gen_imp using mis_var_prop=seq(.1, .8, .1), var_mis_prop=seq(.3, .6, .1), and MNAR cut-offs drawn from U(0.3, 0.6), then impute each degraded dataset with method-specific wrappers: QRILC (impute.QRILC on log-transformed data), GSimp (GS_impute with iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred'), and kNN-TN (imputeKNN with k=3, distance='truncation', perc=0 on log-transformed data). For each imputed dataset, compute TPR = |S ∩ S'| / |S| with Ttest_P_cal_plot, where S is the set of t-test significant variables in the original complete data and S' the set in the imputed data, at p_cut=0.05 and p_cut=0.01, plotted against Miss_Num. Assemble TPR values into a table keyed by imputation method, missing proportion, and p-cutoff, and save as tpr_results.csv together with the two TPR plots.

## Related tools

- **GSimp (GS_impute)** (Gibbs-sampler imputation method under evaluation; wrapper sim_GS_wrapper with iters_each=50, iters_all=10, initial='qrilc', lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred') — https://github.com/WandeRum/GSimp
- **QRILC (impute.QRILC, imputeLCMD)** (Comparison imputation method; wrapper sim_QRILC_wrapper applied on log-transformed data)
- **kNN-TN (Trunc_KNN)** (Comparison imputation method; wrapper sim_trKNN_wrapper with imputeKNN k=3, distance='truncation', perc=0 on log-transformed data) — https://github.com/WandeRum/GSimp
- **glmnet** (Elastic-net prediction model used inside GS_impute during imputation of each degraded dataset)
- **Ttest_P_cal_plot** (Evaluation function computing TPR = |S ∩ S'| / |S| at p_cut=0.05 and p_cut=0.01 and producing TPR-vs-Miss_Num plots) — https://github.com/WandeRum/GSimp

## Examples

```
source('GSimp_evaluation.R'); sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1), cores=1)
```

## Evaluation signals

- TPR curves across increasing Miss_Num should show GSimp achieving the highest overall TPR relative to QRILC and kNN-TN at both p_cut=0.05 and p_cut=0.01
- TPR values fall in [0, 1] and TPR = |S ∩ S'| / |S| is computed against the same reference set S from the original complete data for all methods
- Degraded datasets must span mis_var_prop = 0.1–0.8 and var_mis_prop = 0.3–0.6 with MNAR cut-offs drawn from U(0.3, 0.6), and TPR must be reported for both p-cutoffs
- tpr_results.csv rows must resolve uniquely by (imputation method, missing proportion, p-cutoff)

## Limitations

- MNAR generation and TPR evaluation depend on the specific simulation design (two-group factor, 80 observations per group); parameter choices such as N(0, 0.5^2) and covariance from the diabetes dataset (P=76) are not fully justified
- Reduced iteration counts (iters_all=10, iters_each=50) are acceptable per the authors' tests, but iters_all=20 / iters_each=100 are recommended for Gibbs sampler convergence
- No specific version numbers were reported for R packages glmnet, vegan, ropls, or imputeLCMD

## Evidence

- [abstract] GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity.: "The results show that GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
- [methods] MNAR_gen_imp simulation parameters: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
- [supplementary] Three methods compared in supplementary TPR analysis: "based on three imputation methods: QRILC (green triangle), GSimp (blue square), and kNN-TN (purple cross)"
- [readme] Evaluation pipeline source files in GSimp repo: "GSimp_evaluation.R contains MNAR generation and evaluation functions which are part of our missing value imputation evaluation pipeline."
- [readme] GS_impute default truncation bounds for left-censored data: "Here, lo=-Inf, hi='min' are default setting for
    left-censored missing values where the upper bound is set to the
    minimum value of non-missing part"
