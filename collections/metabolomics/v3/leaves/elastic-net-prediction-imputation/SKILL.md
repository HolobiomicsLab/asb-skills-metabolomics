---
name: elastic-net-prediction-imputation
description: Use when targeted metabolomics datasets (e.g., LC/MS FFAs, bile acids)
  contain left-censored missing values caused by LOD/LOQ (MNAR) and distribution-preserving
  imputation is needed before univariate or multivariate analyses such as PLS-DA.
when_to_use_negative:
- Data are untargeted GC/MS profiling with mostly MCAR/MAR missingness — non-informative
  bounds (+Inf/-Inf) would be needed rather than the left-censored default lo=-Inf,
  hi='min'
- Missing values follow MNAR but a strict upper bound is inappropriate and no LOQ/LOD
  or quantile threshold is available to set hi
- Imputed values must not be stochastic — GSimp draws from a truncated normal distribution,
  unlike deterministic half-minimum (HM) substitution
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: Core R package providing GS_impute and pre_processing_GS_wrapper for Gibbs
    sampler based left-censored imputation
  repo: https://github.com/WandeRum/GSimp
- name: glmnet
  role: Elastic net prediction model (lambda=0.01, alpha=0.5) used within each Gibbs
    iteration to predict candidate values for missing entries
- name: imputeLCMD
  role: Provides impute.QRILC for initializing missing values before the Gibbs sampler
    runs
- name: R
  role: Implementation language; GSimp.R contains the core functions and scripts are
    sourced into an R session
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:c71b25c248b6d02b04735fd0e073a76c481ccd325c47cce2b32f7ff6cc648a8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - R
  - glmnet
  - imputeLCMD
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# elastic-net-prediction-imputation

## Summary

Embeds an elastic net prediction model (glmnet, lambda=0.01, alpha=0.5) inside an iterative Gibbs sampler (GSimp) to impute left-censored MNAR missing values in targeted metabolomics data by sampling from truncated normal distributions bounded by the minimum observed value.

## When to use

Use when targeted metabolomics datasets (e.g., LC/MS FFAs, bile acids) contain left-censored missing values caused by LOD/LOQ (MNAR) and distribution-preserving imputation is needed before univariate or multivariate analyses such as PLS-DA.

## When NOT to use

- Data are untargeted GC/MS profiling with mostly MCAR/MAR missingness — non-informative bounds (+Inf/-Inf) would be needed rather than the left-censored default lo=-Inf, hi='min'
- Missing values follow MNAR but a strict upper bound is inappropriate and no LOQ/LOD or quantile threshold is available to set hi
- Imputed values must not be stochastic — GSimp draws from a truncated normal distribution, unlike deterministic half-minimum (HM) substitution

## Inputs

- Targeted metabolomics data matrix with left-censored missing values (e.g., real_data.csv, targeted_data.csv containing FFAs and BAs)
- Optional truncation bounds per variable (e.g., known LOQ/LOD values or a quantile vector such as sapply(data, function(x) quantile(x, .1, na.rm=T)))

## Outputs

- Fully imputed data matrix with no NA entries, recovered to original scale (scale and exponential recovery)
- GS_impute result object (result$data_imp) tracing the Gibbs sampler at specified missing positions

## How to apply

Pre-process by log-transforming non-normal data, initializing missing entries (QRILC via imputeLCMD, or 'lsym'/'rsym' tail draws), then centralizing and scaling for elastic net prediction. In GS_impute, process missing variables sequentially from least to most missing: fit a glmnet elastic net (lambda=0.01, alpha=0.5) to predict candidate values y_hat, estimate residual standard deviation sigma, and draw imputed values from a truncated normal N(y_hat, sigma^2) with lo=-Inf and hi='min' (minimum observed value; a quantile such as the 10% quantile can relax a too-strict bound). Run the Gibbs sampler for iters_all x iters_each rounds (default 20 x 100; reduced 10 x 50 is acceptable) updating predictions and re-sampling each round, then recover scaling and exponentiate to return the completed matrix. The full sequence is wrapped in pre_processing_GS_wrapper for one-step processing and imputation, and a parallel version (n_cores argument) handles large-scale datasets.

## Related tools

- **GSimp** (Core R package providing GS_impute and pre_processing_GS_wrapper for Gibbs sampler based left-censored imputation) — https://github.com/WandeRum/GSimp
- **glmnet** (Elastic net prediction model (lambda=0.01, alpha=0.5) used within each Gibbs iteration to predict candidate values for missing entries)
- **imputeLCMD** (Provides impute.QRILC for initializing missing values before the Gibbs sampler runs)
- **R** (Implementation language; GSimp.R contains the core functions and scripts are sourced into an R session) — https://github.com/WandeRum/GSimp

## Examples

```
source('GSimp.R'); result <- GS_impute(data_log_sc, iters_each=100, iters_all=20, initial='qrilc', lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred')
```

## Evaluation signals

- Returned matrix contains no NA entries and matches original dimensions after scale_recover and exponential recovery
- Imputed values respect the upper truncation bound (hi='min' or chosen quantile), unlike kNN-TN where imputed values can exceed the truncation point
- A favorable linear trend exists between imputed and original values, as GSimp showed versus QRILC's independent stochastic draws
- Overall data distribution and variances are preserved (no distortion as seen with HM), verified in univariate and multivariate analyses (e.g., PLS-DA) and statistical sensitivity checks
- Settings match paper defaults: lambda=0.01, alpha=0.5, lo=-Inf, hi='min', iters_all=20, iters_each=100 (or validated reduced 10 x 50)

## Limitations

- A large number of iterations (iters_all=20, iters_each=100) is recommended for MCMC convergence; reduced settings (10 x 50) only slightly affect accuracy per simulation tests
- If the minimum observed value is too strict as the upper bound, a quantile (e.g., 10%) or known LOQ/LOD should be used instead; exact threshold settings for different conditions are not fully defined in the paper
- Package versions are pinned in the README (e.g., glmnet 2.0-13, imputeLCMD 2) but the paper reports no specific version numbers for glmnet, vegan, ropls, or imputeLCMD
- Applicability to other -omics or complex scenarios is untested; authors state future evaluation is needed

## Evidence

- [methods] GSimp embeds an elastic net prediction model (glmnet) within an iterative Gibbs sampler: [evidence span withheld — anchor status: not_found]
- [methods] missing values are initialized (e.g., by QRILC): [evidence span withheld — anchor status: not_found]
- [methods] Here, lo=-Inf, hi='min' are default setting for left-censored missing values: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [discussion] a large number of iterations (iters_all = 20, iters_each = 100) are preferable for the convergence of parameters: "a large number of iterations (iters_all = 20, iters_each = 100) are preferable for the convergence of parameters"
- [readme] GS_impute(., iters_each=50, iters_all=10, initial = data_raw_log_qrilc_sc_df, lo=-Inf, hi= 'min', n_cores=2, imp_model='glmnet_pred'): "result <- data_raw_log_sc %>% GS_impute(., iters_each=50, iters_all=10, 
                                             initial = data_raw_log_qrilc_sc_df,"
- [readme] All aboved steps has been wrapped into the pre_processing_GS_wrapper function for a one-step processing and imputation: "All aboved steps has been wrapped into the
*pre\_processing\_GS\_wrapper* function for a one-step processing and
imputation."
