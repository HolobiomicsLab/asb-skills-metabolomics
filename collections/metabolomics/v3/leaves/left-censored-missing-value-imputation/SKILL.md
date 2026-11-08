---
name: left-censored-missing-value-imputation
description: Use when a targeted metabolomics (LC/MS or GC/MS) data matrix contains
  left-censored MNAR missing values caused by LOD/LOQ, and prediction-based imputation
  bounded by the censoring point is needed before downstream univariate or multivariate
  analysis.
when_to_use_negative:
- Missingness is MCAR/MAR (e.g., non-targeted profiling data without missing retrieval)
  unless non-informative bounds lo=-Inf, hi=Inf are deliberately supplied
- Values are right-censored rather than left-censored — use 'rsym' initialization
  with an informative lower truncation point instead of the default left-censored
  configuration
- The dataset is expected to require heavy computation without parallelization — the
  sequential least-to-most-missing ordering and 20x100 iterations can be slow; use
  the parallel version for large-scale datasets
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: core Gibbs sampler imputation framework (GS_impute, pre_processing_GS_wrapper,
    parallel version)
  repo: https://github.com/WandeRum/GSimp
- name: glmnet
  role: elastic net prediction model (lambda=0.01, alpha=0.5) embedded in the Gibbs
    sampler
- name: imputeLCMD
  role: provides impute.QRILC for initializing missing values before sampling
- name: R
  role: implementation environment for GSimp and the evaluation pipeline
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:914523f5e0ab70a744f2edde4555fe0842e0526c0bbbe0080aa4db10cc7af67d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - QRILC
  - R
  - glmnet
  - imputeLCMD
  - kNN-TN
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# left-censored-missing-value-imputation

## Summary

GSimp is a Gibbs sampler based left-censored missing value imputation approach for MNAR missing values in targeted metabolomics data, where an elastic net prediction model (glmnet) is embedded in an iterative Gibbs sampler that draws imputed values from truncated normal distributions bounded by the censoring point.

## When to use

Use when a targeted metabolomics (LC/MS or GC/MS) data matrix contains left-censored MNAR missing values caused by LOD/LOQ, and prediction-based imputation bounded by the censoring point is needed before downstream univariate or multivariate analysis.

## When NOT to use

- Missingness is MCAR/MAR (e.g., non-targeted profiling data without missing retrieval) unless non-informative bounds lo=-Inf, hi=Inf are deliberately supplied
- Values are right-censored rather than left-censored — use 'rsym' initialization with an informative lower truncation point instead of the default left-censored configuration
- The dataset is expected to require heavy computation without parallelization — the sequential least-to-most-missing ordering and 20x100 iterations can be slow; use the parallel version for large-scale datasets

## Inputs

- targeted metabolomics data matrix with missing values (e.g., real_data.csv containing FFAs and BAs, targeted_data.csv for LC/MS, untargeted_data.csv for GC/MS)
- data_sim.csv simulated dataset for evaluation
- optional truncation bounds (LOQ/LOD values or per-variable quantiles) and initialization choice ('qrilc', 'lsym', 'rsym', or a pre-initialized data frame)

## Outputs

- fully imputed data matrix with no NA entries, on the original (un-transformed, un-scaled) scale
- GS_impute result object tracing the Gibbs sampler at specified missing positions
- optionally per-variable NRMSE and Sum of Ranks (SOR) comparison tables against HM, QRILC, and kNN-TN

## How to apply

Log-transform non-normal data, initialize missing entries (default via QRILC from imputeLCMD, or 'lsym'/'rsym' tail-symmetric draws), then centralize and scale for elastic net prediction. Run the GS_impute core function with iters_each=100 and iters_all=20 (defaults; iters_each=50, iters_all=10 acceptable), processing missing variables sequentially from least to most missing. For each variable, fit an elastic net model (glmnet, lambda=0.01, alpha=0.5) to predict candidate values y_hat, estimate residual sigma, and iteratively draw imputations from a truncated normal N(y_hat, sigma^2) with lower bound lo=-Inf and upper bound hi='min' (the minimum observed value of each variable) for left-censored data. Relax hi to a known LOQ or observed-value quantile (e.g. hi=sapply(data, function(x) quantile(x, .1, na.rm=T))) when the minimum is too strict; use lo=-Inf, hi=Inf for MCAR/MAR and 'rsym' initialization with an informative lower bound for right-censored data. Finally recover scaling (scale_recover with method='recover'), exponentiate, and output the completed matrix; pre_processing_GS_wrapper wraps the whole pipeline in one step.

## Related tools

- **GSimp** (core Gibbs sampler imputation framework (GS_impute, pre_processing_GS_wrapper, parallel version)) — https://github.com/WandeRum/GSimp
- **glmnet** (elastic net prediction model (lambda=0.01, alpha=0.5) embedded in the Gibbs sampler)
- **imputeLCMD** (provides impute.QRILC for initializing missing values before sampling)
- **R** (implementation environment for GSimp and the evaluation pipeline) — https://github.com/WandeRum/GSimp

## Examples

```
source('GSimp.R'); result <- data_raw_log_sc %>% GS_impute(., iters_each=50, iters_all=10, initial=data_raw_log_qrilc_sc_df, lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred')
```

## Evaluation signals

- Returned matrix contains no NA entries and matches the paper defaults (lambda=0.01, alpha=0.5, lo=-Inf, hi='min')
- Imputed values respect the truncation bound: no imputed value exceeds the minimum observed value of its variable (the failure mode observed for kNN-TN)
- On MNAR-degraded data (MNAR_gen_imp, mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1)), GSimp attains the lowest NRMSE-based Sum of Ranks versus HM, QRILC, and kNN-TN
- A favorable linear trend holds between imputed and original values, and the imputed distribution matches the observed left tail
- Downstream PLS-DA / univariate results are not distorted relative to the complete-data reference

## Limitations

- HM-style substitution can severely distort the data distribution; GSimp's advantage is specific to left-censored MNAR structure
- R package versions (glmnet 2.0-13, imputeLCMD 2, etc.) are pinned in the README; no version guidance was given in the discussion
- Exact truncation-point settings 'for different conditions' (10% quantile vs known LOQ/LOD) are not fixed; quantile or LOQ bounds are only qualitatively validated as comparable to the default minimum bound
- Extension to other -omics scenarios remains untested per the authors

## Evidence

- [intro] GSimp embeds elastic net in Gibbs sampler: "we developed a left-censored missing value imputation framework, GSimp, where a prediction model was embedded in an iterative Gibbs sampler"
- [methods] default truncation bounds: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [readme] quantile upper bounds: "quantile values can be applied if minimum is too strict. For example, hi=sapply(data, function(x) quantile(x, .1, na.rm=T)) represent the 10% quantiles"
- [discussion] iteration defaults: "a large number of iterations (iters_all = 20, iters_each = 100) are preferable for the convergence of parameters"
- [discussion] sequential ordering: "we applied a sequential imputation procedure for missing variables from the least number of missing values to the most"
- [abstract] MNAR motivation: "Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics studies"
- [discussion] bounded imputation advantage: "ensured a favorable linear trend between imputed and original values as well as a reasonable bound for the imputed values"
