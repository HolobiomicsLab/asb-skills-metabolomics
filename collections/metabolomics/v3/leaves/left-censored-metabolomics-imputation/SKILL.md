---
name: left-censored-metabolomics-imputation
description: Use when targeted metabolomics datasets contain left-censored missing
  values caused by LOD/LOQ (MNAR) and a robust imputation method is needed before
  univariate t-tests, PLS-DA, or differential-variable detection.
when_to_use_negative:
- Missingness is MCAR/MAR rather than left-censored MNAR — with non-informative bounds
  (+∞, -∞) GSimp extends to MCAR/MAR, but the default lo=-Inf/hi='min' setting targets
  left-censored values.
- Data still contains raw un-retrieved missing values in non-targeted GC/MS studies
  — the paper recommends missing retrieval from raw spectral data first, assuming
  the majority of missing values are MCAR/MAR before manual retrieval.
- Upper truncation is stricter than needed — when the minimum is too strict, quantile
  bounds (e.g. hi=sapply(data, function(x) quantile(x, .1, na.rm=T))) or known LOQ/LOD
  values should be used instead.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: GSimp
  role: Core Gibbs sampler imputation method (GS_impute) plus pre_processing_GS_wrapper
    for one-step pre-processing and imputation
  repo: https://github.com/WandeRum/GSimp
- name: QRILC
  role: Baseline imputation method (impute.QRILC from imputeLCMD) used for initialization
    and for comparison wrappers
- name: kNN-TN
  role: Baseline truncated-normal kNN imputation (imputeKNN, k=3, distance='truncation',
    perc=0) for comparison
- name: glmnet
  role: Elastic-net prediction model embedded in the Gibbs sampler (imp_model='glmnet_pred')
- name: R
  role: Implementation and evaluation pipeline environment; packages include imputeLCMD
    2, glmnet 2.0-13, doParallel 1.0.11, vegan 2.4-5, ropls 1.8.0
  repo: https://github.com/WandeRum/GSimp
- name: MVI-evaluation
  role: Imputation evaluation pipeline (missing generation, imputations, evaluations)
    with Impute_wrapper.R for adding XXX_wrapper methods
  repo: https://github.com/WandeRum/MVI-evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:4b2f35c5d93b85af9bfae8163f5e3fd58a04aee11cf5db2ba006e9246dbb998d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - MVI-evaluation
  - QRILC
  - R
  - glmnet
  - imputeLCMD
  - kNN-TN
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 21
---

# left-censored-metabolomics-imputation

## Summary

GSimp is an iterative Gibbs sampler based imputation approach for left-censored (MNAR) missing values in targeted metabolomics data, where an elastic-net prediction model (glmnet) is embedded in the Gibbs sampler. It is compared against QRILC and kNN-TN using an imputation evaluation pipeline covering accuracy, distribution, univariate/multivariate analyses, and statistical sensitivity.

## When to use

Use when targeted metabolomics datasets contain left-censored missing values caused by LOD/LOQ (MNAR) and a robust imputation method is needed before univariate t-tests, PLS-DA, or differential-variable detection.

## When NOT to use

- Missingness is MCAR/MAR rather than left-censored MNAR — with non-informative bounds (+∞, -∞) GSimp extends to MCAR/MAR, but the default lo=-Inf/hi='min' setting targets left-censored values.
- Data still contains raw un-retrieved missing values in non-targeted GC/MS studies — the paper recommends missing retrieval from raw spectral data first, assuming the majority of missing values are MCAR/MAR before manual retrieval.
- Upper truncation is stricter than needed — when the minimum is too strict, quantile bounds (e.g. hi=sapply(data, function(x) quantile(x, .1, na.rm=T))) or known LOQ/LOD values should be used instead.

## Inputs

- data_sim.csv — simulated dataset generated per simulation_generation.Rmd
- real_data.csv — real-world targeted metabolomics dataset (FFAs and BAs)
- targeted_data.csv — targeted LC/MS data with missing values
- untargeted_data.csv — untargeted GC/MS data after manual missing retrieval
- data matrix (samples × metabolites) with left-censored NA positions

## Outputs

- completed imputed data matrix (after scaling and exponential recovery)
- tpr_results.csv — TPR table keyed by imputation method, missing proportion, and p-cutoff
- TPR curves vs Miss_Num comparing GSimp, QRILC, and kNN-TN

## How to apply

Pre-process the data by log-transforming non-normal values, initializing missing entries with QRILC (impute.QRILC), then centralizing and scaling for the elastic-net prediction model. Run GS_impute with iters_each=50, iters_all=10 (defaults of 100/20 are preferable for MCMC convergence), initial='qrilc', lo=-Inf and hi='min' — the upper bound set to the minimum of the non-missing part, the default for left-censored data — and imp_model='glmnet_pred', n_cores=2 for the parallel version. After imputation, recover scale and exponential transforms before output. For evaluation, generate MNAR-degraded data with MNAR_gen_imp (mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1), cut-off from U(0.3, 0.6)) and compare against sim_QRILC_wrapper and sim_trKNN_wrapper (k=3, distance='truncation', perc=0). Assess statistical sensitivity by computing TPR = |S ∩ S'| / |S| with Ttest_P_cal_plot at p_cut=0.05 and 0.01, comparing significant variables in the complete data (S) versus imputed data (S').

## Related tools

- **GSimp** (Core Gibbs sampler imputation method (GS_impute) plus pre_processing_GS_wrapper for one-step pre-processing and imputation) — https://github.com/WandeRum/GSimp
- **QRILC** (Baseline imputation method (impute.QRILC from imputeLCMD) used for initialization and for comparison wrappers)
- **kNN-TN** (Baseline truncated-normal kNN imputation (imputeKNN, k=3, distance='truncation', perc=0) for comparison)
- **glmnet** (Elastic-net prediction model embedded in the Gibbs sampler (imp_model='glmnet_pred'))
- **R** (Implementation and evaluation pipeline environment; packages include imputeLCMD 2, glmnet 2.0-13, doParallel 1.0.11, vegan 2.4-5, ropls 1.8.0) — https://github.com/WandeRum/GSimp
- **MVI-evaluation** (Imputation evaluation pipeline (missing generation, imputations, evaluations) with Impute_wrapper.R for adding XXX_wrapper methods) — https://github.com/WandeRum/MVI-evaluation

## Examples

```
result <- GS_impute(data_raw_log_sc, iters_each=50, iters_all=10, initial=data_raw_log_qrilc_sc_df, lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred')
```

## Evaluation signals

- GSimp should show the overall highest TPR across different numbers of missing variables at both p_cut=0.05 and p_cut=0.01 relative to QRILC and kNN-TN.
- Imputed values should show a favorable linear trend between original and imputed values with a reasonable bound (no right shift beyond the truncation point, unlike kNN-TN).
- The overall data distribution and variances should be preserved (unlike HM, which severely distorts the distribution).
- Convergence diagnostics: larger iters_all=20 and iters_each=100 should give similar accuracy to iters_all=10/iters_each=50 on simulation data (SOR checked along different numbers of missing variables).
- NA positions in the output must match the original missing positions (NA_pos re-introduced after scaling/initialization before GS_impute).

## Limitations

- A large number of iterations (iters_all=20, iters_each=100) is recommended for MCMC convergence, though smaller settings (10/50) were tested not to severely affect accuracy on simulation data.
- GSimp's applicability to 'other -omics' and more complex scenarios was not evaluated; the authors state future evaluation is needed.
- Exact threshold/quantile settings for different conditions (e.g., 10% quantile vs known LOQ/LOD) are not predefined and must be chosen by the practitioner.
- No specific version numbers were reported for glmnet, vegan, ropls, or imputeLCMD in the paper (versions are only listed in the README).
- Minimum as the upper bound can be too strict; quantile values (e.g., 10% quantile) may need to be applied instead.

## Evidence

- [abstract] We have developed an iterative Gibbs sampler based left-censored missing value imputation approach (GSimp).: "We have developed an iterative Gibbs sampler based left-censored missing value imputation approach (GSimp)."
- [methods] GS_impute default bounds: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [methods] MNAR generation parameters: "sim_MNAR_list <- MNAR_gen_imp(data_c=data_sim_sc, mis_var_prop=seq(.1, .8, .1),"
- [readme] pre-processing wrapper: [evidence span withheld — anchor status: not_found]
- [abstract] MNAR motivation: "Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics studies and could be recognized as missing not at"
- [abstract] GSimp performance: "The results show that GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
