---
name: gibbs-sampler-imputation-parameterization
description: Use when a targeted metabolomics intensity matrix (e.g. FFA/BA LC-MS
  data) contains left-censored missing values attributed to LOD/LOQ (MNAR) and an
  iterative Gibbs sampler must be configured — iteration counts, initialization method,
  and upper/lower truncation bounds — before running GS_impute.
when_to_use_negative:
- Data are GC-MS untargeted profiles where missingness is MCAR/MAR (e.g. before manual
  missing retrieval) — set non-informative ±∞ bounds rather than the left-censored
  defaults, or the sampler will force imputed values below the observed minimum.
- A non-MetaboAnalyst pipeline (kNN, SVD, PPCA, BPCA, RF) is already chosen and the
  goal is not left-censored MNAR imputation — the parameterization here is specific
  to GSimp's Gibbs sampler.
- The matrix is not log-transformed normal data — running the sampler on raw non-normal
  intensities without the pre-processing wrapper distorts the truncated-normal assumptions.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0602
tools:
- name: GSimp
  role: Core Gibbs sampler imputation via GS_impute; parameterized with iters_each,
    iters_all, initial, lo, hi, n_cores, imp_model
  repo: https://github.com/WandeRum/GSimp
- name: QRILC (imputeLCMD)
  role: Default initialization method 'qrilc' via impute.QRILC before the Gibbs sampler
    runs
- name: glmnet
  role: Elastic-net prediction model used inside GS_impute (imp_model='glmnet_pred')
- name: R
  role: Runtime for GSimp.R, GSimp_evaluation.R, Impute_wrapper.R and the vignette
    scripts
  repo: https://github.com/WandeRum/GSimp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:a0afef6a33e07855eaaa1f0410e9268765432b73fdb9cd52f2825669886bef61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - QRILC
  - QRILC (imputeLCMD)
  - R
  - glmnet
  - kNN-TN
  - vegan
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# gibbs-sampler-imputation-parameterization

## Summary

Parameterization and invocation of the GSimp iterative Gibbs sampler for left-censored (MNAR) missing values in targeted metabolomics matrices: log-transform, QRILC initialization, scale, impute with lo=-Inf / hi='min' truncation bounds, then recover scale and exponentiate. It encodes the iteration counts, initialization options, and truncation-point choices that the article tested on FFA, BA, and simulated diabetes data (160 x 76).

## When to use

Use when a targeted metabolomics intensity matrix (e.g. FFA/BA LC-MS data) contains left-censored missing values attributed to LOD/LOQ (MNAR) and an iterative Gibbs sampler must be configured — iteration counts, initialization method, and upper/lower truncation bounds — before running GS_impute.

## When NOT to use

- Data are GC-MS untargeted profiles where missingness is MCAR/MAR (e.g. before manual missing retrieval) — set non-informative ±∞ bounds rather than the left-censored defaults, or the sampler will force imputed values below the observed minimum.
- A non-MetaboAnalyst pipeline (kNN, SVD, PPCA, BPCA, RF) is already chosen and the goal is not left-censored MNAR imputation — the parameterization here is specific to GSimp's Gibbs sampler.
- The matrix is not log-transformed normal data — running the sampler on raw non-normal intensities without the pre-processing wrapper distorts the truncated-normal assumptions.

## Inputs

- Raw targeted metabolomics data matrix with missing values (e.g. real_data.csv, targeted_data.csv — samples x variables)
- Scaled simulated matrix (data_sim.csv, 160 samples x 76 variables) for evaluation
- Known LOQ/LOD thresholds or a desired quantile (e.g. 10%) to serve as upper truncation points, when available

## Outputs

- Complete log-scaled intermediate matrix data_imp_log_sc from GS_impute
- Final imputed matrix in the original scale (after scale recovery and exponential back-transform)
- Imputation metadata: NA_pos indices and scale/scale-recovery parameter table data_raw_log_qrilc_sc_df_param

## How to apply

Wrap the steps in pre_processing_GS_wrapper: (1) log-transform the raw matrix; (2) initialize missing entries with impute.QRILC; (3) centralize and scale via scale_recover(method='scale'); (4) re-introduce NAs at the original missing positions; (5) call GS_impute with iters_each=50, iters_all=10 for quick runs (default iters_each=100, iters_all=20 for full convergence), initial=data_raw_log_qrilc_sc_df, lo=-Inf, hi='min', and imp_model='glmnet_pred'; (6) recover scale with scale_recover(method='recover', param_df=…) and exponentiate. The hi='min' bound sets the upper truncation point to each variable's minimum observed value; quantiles (e.g. 10% via sapply(data, function(x) quantile(x, .1, na.rm=T))) or known LOQ/LOD values can replace it when the minimum is too strict. Sequential imputation goes from the least-missing variable to the most-missing. Set +Inf/-Inf bounds instead when missingness is MCAR/MAR rather than MNAR.

## Related tools

- **GSimp** (Core Gibbs sampler imputation via GS_impute; parameterized with iters_each, iters_all, initial, lo, hi, n_cores, imp_model) — https://github.com/WandeRum/GSimp
- **QRILC (imputeLCMD)** (Default initialization method 'qrilc' via impute.QRILC before the Gibbs sampler runs)
- **glmnet** (Elastic-net prediction model used inside GS_impute (imp_model='glmnet_pred'))
- **R** (Runtime for GSimp.R, GSimp_evaluation.R, Impute_wrapper.R and the vignette scripts) — https://github.com/WandeRum/GSimp

## Examples

```
data_imp <- data_raw_log_sc %>% GS_impute(., iters_each=50, iters_all=10, initial=data_raw_log_qrilc_sc_df, lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred')
```

## Evaluation signals

- Re-scaled/recovered matrix should have the same dimensions as the input and no remaining NAs at the recorded NA_pos.
- iters_all=10, iters_each=50 runs should show imputation accuracy close to the iters_all=20, iters_each=100 default on the simulation data (article reports smaller iterations 'won't severely affect the imputation accuracy').
- Every imputed value in a left-censored run must be <= the upper truncation point (hi), e.g. each variable's minimum observed value or chosen quantile.
- PCA/Procrustes comparison (2 PCs via vegan) of imputed vs original ordination should show the lowest sum of squared errors for GSimp versus QRILC and kNN-TN wrappers.
- Scaling-recovery round trip (scale_recover with method='scale' then method='recover' with saved param_df) must reproduce the original column means and standard deviations before exponentiation.

## Limitations

- Smaller iteration counts (iters_all=10, iters_each=50) trade MCMC convergence for speed; the article recommends iters_all=20, iters_each=100 for full convergence.
- hi='min' can be too strict a truncation point; the article suggests 10% quantiles or known LOQ/LOD as alternatives, but no universal default is prescribed.
- GSimp's left-censored defaults assume MNAR from LOD/LOQ in LC-MS targeted metabolomics; behaviour on MCAR/MAR or on GC-MS post-missing-retrieval data was only conditionally validated, not exhaustively tested.
- Parameter ranges for the simulated data (N(0, 0.5^2) means, SVD-derived covariance from a diabetes dataset of P=76) are described but not fully justified in the discussion.

## Evidence

- [methods] Log-transformation (for non-normal data): "-   Log-transformation (for non-normal data)"
- [methods] GS_impute core function: "The function *GS_impute* is the core function for the imputation of missing data and tracing the Gibbs sampler with certain missing positions."
- [methods] Truncation bound defaults: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [readme] Iteration-count recommendation: [evidence span withheld — anchor status: not_found]
- [readme] Quantile upper-bound alternative: "hi=sapply(data, function(x) quantile(x, .1, na.rm=T)) represent the 10% quantiles of each variable are set to the upper bounds"
- [readme] Wrapper pipeline: [evidence span withheld — anchor status: not_found]
- [abstract] GSimp outperforms other methods: "The results show that GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
- [discussion] Parallel version for large datasets: "we additionally implemented a parallel version which can run Gibbs sampler on multiple missing variables simultaneously"
