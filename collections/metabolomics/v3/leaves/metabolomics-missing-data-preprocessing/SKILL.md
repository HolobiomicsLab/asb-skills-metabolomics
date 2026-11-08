---
name: metabolomics-missing-data-preprocessing
description: Use when a targeted metabolomics data matrix (e.g., LC/MS FFA/BA data)
  contains left-censored MNAR missing values caused by LOD/LOQ and a complete matrix
  is needed before PLS-DA or other analyses.
when_to_use_negative:
- The input matrix has no missing values — the pre-processing/Gibbs chain is unnecessary
  and QRILC initialization would fabricate nothing meaningful
- Missingness is MCAR/MAR (e.g., non-targeted GC/MS data before missing retrieval)
  and left-censored truncation hi='min' is inappropriate, unless non-informative bounds
  (+Inf/-Inf) are deliberately supplied
- Missing values are right-censored rather than left-censored — the 'rsym' initialization
  path exists, but the default lo=-Inf, hi='min' bounds target left-censoring
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: Core R implementation providing GS_impute (Gibbs sampler imputation), scale_recover,
    and pre_processing_GS_wrapper
  repo: https://github.com/WandeRum/GSimp
- name: glmnet
  role: Elastic net prediction model (lambda=0.01, alpha=0.5) used within each Gibbs
    iteration to predict candidate values for missing entries
- name: imputeLCMD
  role: Provides impute.QRILC for initial imputation of missing values before Gibbs
    sampling
- name: doParallel
  role: Supports the parallel GSimp version (n_cores argument) for large-scale metabolomics
    datasets
- name: R
  role: Runtime environment for GSimp, its evaluation pipeline, and vignette
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:2487cae266fe5ed5c0d0b50e3c818710161ccab11c69081874c7cdaba166d301
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - R
  - doParallel
  - glmnet
  - imputeLCMD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolomics-missing-data-preprocessing

## Summary

A left-censored missing value imputation workflow for targeted metabolomics that embeds a glmnet elastic net prediction model in an iterative Gibbs sampler (GSimp), preceded by log-transformation, QRILC initialization, and scaling, and followed by scaling/exponential recovery.

## When to use

Use when a targeted metabolomics data matrix (e.g., LC/MS FFA/BA data) contains left-censored MNAR missing values caused by LOD/LOQ and a complete matrix is needed before PLS-DA or other analyses.

## When NOT to use

- The input matrix has no missing values — the pre-processing/Gibbs chain is unnecessary and QRILC initialization would fabricate nothing meaningful
- Missingness is MCAR/MAR (e.g., non-targeted GC/MS data before missing retrieval) and left-censored truncation hi='min' is inappropriate, unless non-informative bounds (+Inf/-Inf) are deliberately supplied
- Missing values are right-censored rather than left-censored — the 'rsym' initialization path exists, but the default lo=-Inf, hi='min' bounds target left-censoring

## Inputs

- Targeted metabolomics data matrix with left-censored missing values (e.g., real_data.csv combining FFAs and BAs, targeted_data.csv from LC/MS)
- Optional initialization matrix (QRILC-imputed or pre-initialized data frame)
- Optional per-variable truncation bounds (e.g., known LOQ/LOD values or 10% quantiles via sapply(data, function(x) quantile(x, .1, na.rm=T)))

## Outputs

- Fully imputed data matrix with no NA entries, recovered to original scale (scale and exponential recovery)
- GS_impute result object (result$data_imp) tracing the Gibbs sampler at missing positions

## How to apply

Log-transform the raw data (for non-normal data), initialize missing entries with QRILC via imputeLCMD (or 'lsym'/'rsym' tail draws), then centralize and scale for elastic net prediction. Feed the initialized, NA-reintroduced data to GS_impute, which processes missing variables sequentially from the least to the most missing: for each variable it fits an elastic net model (glmnet, lambda=0.01, alpha=0.5), estimates residual sigma, and draws imputed values from a truncated normal N(y_hat, sigma^2) bounded by lo=-Inf and hi='min' (minimum of observed values; a 10% quantile can relax strictness). Run iters_all=20 x iters_each=100 Gibbs iterations by default (iters_all=10, iters_each=50 acceptable with minor accuracy loss), then recover scale via scale_recover and exponentiate back. All steps are wrapped in pre_processing_GS_wrapper for one-step execution.

## Related tools

- **GSimp** (Core R implementation providing GS_impute (Gibbs sampler imputation), scale_recover, and pre_processing_GS_wrapper) — https://github.com/WandeRum/GSimp
- **glmnet** (Elastic net prediction model (lambda=0.01, alpha=0.5) used within each Gibbs iteration to predict candidate values for missing entries)
- **imputeLCMD** (Provides impute.QRILC for initial imputation of missing values before Gibbs sampling)
- **doParallel** (Supports the parallel GSimp version (n_cores argument) for large-scale metabolomics datasets)
- **R** (Runtime environment for GSimp, its evaluation pipeline, and vignette) — https://github.com/WandeRum/GSimp

## Examples

```
result <- data_raw_log_sc %>% GS_impute(., iters_each=50, iters_all=10, initial=data_raw_log_qrilc_sc_df, lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred')
```

## Evaluation signals

- The returned matrix is fully imputed — no NA entries remain (check is.na(data_imp) sums to zero)
- Imputed values respect truncation bounds: with lo=-Inf, hi='min', no imputed value exceeds the minimum observed value of its variable
- A favorable linear trend exists between imputed and original values, unlike HM (distribution distortion) or kNN-TN (values exceeding the truncation point)
- Gibbs parameters match paper defaults: iters_all=20, iters_each=100 (or reduced 10/50), lambda=0.01, alpha=0.5
- Scaling and exponential recovery reproduce the original data scale — scale_recover(param_df=...) followed by exp() returns log-scale results to raw intensities

## Limitations

- Designed for left-censored MNAR missing values from LOD/LOQ; for MCAR/MAR situations GSimp must be run with non-informative bounds (+Inf/-Inf) instead of the default truncation
- Convergence of MCMC parameters requires a large number of iterations (iters_all=20, iters_each=100); fewer iterations slightly reduce accuracy
- Known LOQ/LOD values or quantile upper bounds (e.g., 10% quantile) should be supplied per condition when the minimum observed value is too strict — exact threshold settings for different conditions are not defined in the paper
- No specific version numbers for glmnet, vegan, ropls, or imputeLCMD are reported in the article (only the README dependency table lists versions such as glmnet 2.0-13, imputeLCMD 2)
- Extension to other -omics or complex scenarios remains untested per the authors

## Evidence

- [readme] All aboved steps has been wrapped into the *pre\_processing\_GS\_wrapper* function for a one-step processing and imputation.: "All aboved steps has been wrapped into the
*pre\_processing\_GS\_wrapper* function for a one-step processing and
imputation."
- [methods] Here, lo=-Inf, hi='min' are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part: "Here, lo=-Inf, hi=‘min’ are default setting for
left-censored missing values where the upper bound is set to the
minimum value of non-missing part"
- [discussion] we applied a sequential imputation procedure for missing variables from the least number of missing values to the most: "we applied a sequential imputation procedure for missing variables from the least number of missing values to the most"
- [discussion] a large number of iterations (iters_all = 20, iters_each = 100) are preferable for the convergence of parameters: "a large number of iterations (iters_all = 20, iters_each = 100) are preferable for the convergence of parameters"
- [methods] Initialization for missing values (e.g., QRILC): "-   Initialization for missing values (e.g., QRILC)"
