---
name: truncated-normal-sampling
description: Use when a targeted metabolomics (MS-based) data matrix contains missing-not-at-random
  values caused by LOD/LOQ, and imputations must respect a lower-tail truncation bound.
when_to_use_negative:
- Missingness is MCAR/MAR (e.g., untargeted GC/MS data before missing retrieval) with
  no informative truncation bound; non-informative +/-Inf bounds would be needed instead
- Missing values are right-censored (upper-tail), where the 'rsym' initialization
  and upper-truncation setup apply rather than the lo=-Inf, hi='min' defaults
- A single deterministic substitution (e.g., half-minimum) is acceptable, since truncated
  sampling is designed to avoid the distribution distortion deterministic methods
  cause
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: GSimp
  role: Provides GS_impute, the core function that traces the Gibbs sampler and draws
    missing values from the truncated normal distribution
  repo: https://github.com/WandeRum/GSimp
- name: glmnet
  role: Elastic net prediction model (lambda=0.01, alpha=0.5) supplying y_hat candidate
    values for each missing variable before truncated sampling
- name: imputeLCMD
  role: Supplies QRILC initialization of missing entries that seeds the Gibbs sampler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:3ae188fba2b5b025ddc847ecad05d7d2c3e384b6a7edd64f03d5d46fe7d81105
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - R
  - glmnet
  - imputeLCMD
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# truncated-normal-sampling

## Summary

Draws left-censored missing values in targeted metabolomics data from a truncated normal distribution centered on elastic net predictions, embedded in the GSimp iterative Gibbs sampler.

## When to use

Use when a targeted metabolomics (MS-based) data matrix contains missing-not-at-random values caused by LOD/LOQ, and imputations must respect a lower-tail truncation bound.

## When NOT to use

- Missingness is MCAR/MAR (e.g., untargeted GC/MS data before missing retrieval) with no informative truncation bound; non-informative +/-Inf bounds would be needed instead
- Missing values are right-censored (upper-tail), where the 'rsym' initialization and upper-truncation setup apply rather than the lo=-Inf, hi='min' defaults
- A single deterministic substitution (e.g., half-minimum) is acceptable, since truncated sampling is designed to avoid the distribution distortion deterministic methods cause

## Inputs

- Targeted metabolomics data matrix (e.g., real_data.csv, targeted LC/MS data) with left-censored missing values
- Initial imputed data frame (e.g., QRILC output from impute.QRILC on log-transformed data)
- Truncation bounds (lo, hi) such as -Inf and 'min', or per-variable quantile/LOQ/LOD values
- Iteration counts iters_each and iters_all

## Outputs

- Fully imputed data matrix with no NA entries
- Trace of the Gibbs sampler at specified missing positions (from GS_impute)

## How to apply

First pre-process the data (log-transformation for non-normal data, QRILC initialization, centralization and scaling) and re-introduce NA at the original missing positions. For each missing variable — processed sequentially from the least number of missing values to the most — fit an elastic net prediction model with glmnet (lambda=0.01, alpha=0.5) to obtain candidate values y_hat for the missing entries, then estimate the residual standard deviation sigma. Sample imputed values from a truncated normal N(y_hat, sigma^2) with lower bound lo=-Inf and upper bound hi='min' (the minimum non-missing value of the variable); quantiles may substitute a strict minimum, e.g. hi=sapply(data, function(x) quantile(x, .1, na.rm=T)). Iterate the Gibbs sampler iters_all=20 times over iters_each=100 sweeps, re-estimating predictions and re-sampling each round, then recover scaling and exponentiation to return the completed matrix. Rationale: truncation bounds imputed values below the censoring limit while the sequential sampler preserves linear trend between imputed and observed values.

## Related tools

- **GSimp** (Provides GS_impute, the core function that traces the Gibbs sampler and draws missing values from the truncated normal distribution) — https://github.com/WandeRum/GSimp
- **glmnet** (Elastic net prediction model (lambda=0.01, alpha=0.5) supplying y_hat candidate values for each missing variable before truncated sampling)
- **imputeLCMD** (Supplies QRILC initialization of missing entries that seeds the Gibbs sampler)

## Examples

```
result <- GS_impute(data_raw_log_sc, iters_each=50, iters_all=10, initial=data_raw_log_qrilc_sc_df, lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred')
```

## Evaluation signals

- Returned matrix contains no NA entries after scale and exponential recovery
- Every imputed value lies at or below the variable's minimum observed value when hi='min' (truncation bound respected)
- Imputed vs. original values exhibit a favorable linear trend without a right shift past the truncation point
- Sampler run with defaults iters_all=20, iters_each=100 (reduced 10/50 acceptable without severe accuracy loss)
- Variables were imputed in sequential order from least to most missing values

## Limitations

- A large number of iterations (iters_all=20, iters_each=100) is recommended for MCMC convergence; smaller settings may reduce accuracy
- hi='min' can be too strict as an upper bound; quantile values (e.g., the 10% quantile) or known LOQ/LOD can be substituted, but exact per-condition settings are not specified in the paper
- No specific version numbers reported for glmnet or imputeLCMD in the article
- No quantitative runtime or convergence figures given in the main text (deferred to supplementary S1 Table and S2 Fig)

## Evidence

- [abstract] We have developed an iterative Gibbs sampler based left-censored missing value imputation approach (GSimp).: "We have developed an iterative Gibbs sampler based left-censored missing value imputation approach (GSimp)."
- [intro] Quantile regression approach for left-censored missing (QRILC) imputes missing data using random draws from a truncated distribution: "Quantile regression approach for left-censored missing (QRILC) imputes missing data using random draws from a truncated distribution"
- [methods] Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part: "Here, lo=-Inf, hi=‘min’ are default setting for
left-censored missing values where the upper bound is set to the
minimum value of non-missing part"
- [discussion] we applied a sequential imputation procedure for missing variables from the least number of missing values to the most: "we applied a sequential imputation procedure for missing variables from the least number of missing values to the most"
- [readme] **lo** is the lower limits (default=‘-Inf’) and **hi** (default=‘min’) is the upper limits for missing values.: "**lo** is the lower limits (default=‘-Inf’) and **hi**
(default=‘min’) is the upper limits for missing values."
- [readme] hi=sapply(data, function(x) quantile(x, .1, na.rm=T)) represent the 10% quantiles of each variable are set to the upper bounds: "For example, hi=sapply(data,
    function(x) quantile(x, .1, na.rm=T)) represent the 10% quantiles of
    each variable are set to the upper bounds"
