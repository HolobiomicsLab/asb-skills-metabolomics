---
name: truncation-bound-configuration-gibbs-sampler
description: Use when rerunning GSimp on targeted metabolomics data (e.g. FFAs/BAs
  targeted LC/MS matrices) where the default 'min' upper bound is too strict, or when
  the missingness mechanism is right-censored (needs rsym + lower bound) or MCAR/MAR
  (needs +/-Inf bounds).
when_to_use_negative:
- Data are genuinely MCAR/MAR with no censoring structure — use non-informative lo=-Inf,
  hi=Inf bounds rather than a minimum- or quantile-based upper truncation point.
- The dataset is untargeted GC/MS data before manual missing retrieval, where the
  article assumed MCAR/MAR rather than left-censored MNAR.
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: Core Gibbs sampler imputation whose GS_impute function accepts lo/hi truncation
    bounds, initial='rsym'/'qrilc'/'lsym', and the glmnet_pred elastic-net prediction
    model.
  repo: https://github.com/WandeRum/GSimp
- name: imputeLCMD
  role: Provides impute.QRILC used to initialize missing values on the log-scaled
    data before GS_impute is run with alternative bounds.
- name: glmnet
  role: Elastic-net prediction model (imp_model='glmnet_pred') used inside the Gibbs
    sampler iterations.
- name: R
  role: Runtime environment for the GSimp pipeline, including sapply/quantile bound
    computation and CSV output of imputed matrices.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:f6741bca5830890d0fd88d61d8767055fa372e03bc24c8a46542832ee209e190
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
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# truncation-bound-configuration-gibbs-sampler

## Summary

Configure the lo/hi truncation bounds of the GS_impute Gibbs sampler for MNAR left-censored metabolomics data — replacing the default lower=-Inf/upper='min' bounds with known LOQ/LOD values, observed-value quantiles (e.g. 10%), an informative lower bound for right-censored data (with 'rsym' initialization), or +/-Inf bounds for MCAR/MAR.

## When to use

Use when rerunning GSimp on targeted metabolomics data (e.g. FFAs/BAs targeted LC/MS matrices) where the default 'min' upper bound is too strict, or when the missingness mechanism is right-censored (needs rsym + lower bound) or MCAR/MAR (needs +/-Inf bounds).

## When NOT to use

- Data are genuinely MCAR/MAR with no censoring structure — use non-informative lo=-Inf, hi=Inf bounds rather than a minimum- or quantile-based upper truncation point.
- The dataset is untargeted GC/MS data before manual missing retrieval, where the article assumed MCAR/MAR rather than left-censored MNAR.

## Inputs

- Targeted LC/MS metabolomics data matrix with missing values (CSV, e.g. targeted_data.csv or real_data.csv with FFAs and BAs)
- Known LOQ/LOD values per metabolite (optional, when available)
- MNAR-degraded matrix produced by MNAR_gen_imp for accuracy benchmarking

## Outputs

- Imputed data matrix at original scale (CSV) for each truncation configuration
- NRMSE and sum-of-ranks (SOR) comparison table across truncation settings and missingness levels (e.g. truncation_comparison_results.csv)

## How to apply

Load the targeted LC/MS matrix (e.g. targeted_data.csv), log-transform, initialize missing values with QRILC via impute.QRILC, then centralize and scale with scale_recover before imputation. Call GS_impute with iters_each=50, iters_all=10 and imp_model='glmnet_pred', supplying alternative hi bounds such as a known LOQ/LOD value or the observed quantiles: hi=sapply(data, function(x) quantile(x, .1, na.rm=T)). For right-censored missingness, use initial='rsym' with an informative lower truncation bound; for MCAR/MAR, set both lo=-Inf and hi=Inf. Impute in sequence from the least-missing to the most-missing variables, then recover scale and exponential back to the original measurement scale and export the imputed matrix. Compare configurations by computing NRMSE on the log-scale imputed elements of a MNAR-degraded matrix (MNAR_gen_imp with mis_var_prop=seq(.1,.8,.1), var_mis_prop=seq(.3,.6,.1)) and aggregating into a sum-of-ranks (SOR) across missingness levels.

## Related tools

- **GSimp** (Core Gibbs sampler imputation whose GS_impute function accepts lo/hi truncation bounds, initial='rsym'/'qrilc'/'lsym', and the glmnet_pred elastic-net prediction model.) — https://github.com/WandeRum/GSimp
- **imputeLCMD** (Provides impute.QRILC used to initialize missing values on the log-scaled data before GS_impute is run with alternative bounds.)
- **glmnet** (Elastic-net prediction model (imp_model='glmnet_pred') used inside the Gibbs sampler iterations.)
- **R** (Runtime environment for the GSimp pipeline, including sapply/quantile bound computation and CSV output of imputed matrices.)

## Examples

```
result <- GS_impute(data_raw_log_sc, iters_each=50, iters_all=10, initial=data_raw_log_qrilc_sc_df, lo=-Inf, hi=sapply(data, function(x) quantile(x, .1, na.rm=T)), n_cores=2, imp_model='glmnet_pred')
```

## Evaluation signals

- NRMSE computed on the log-scale values of the imputed missing elements falls in a comparable range to the default lo=-Inf/hi='min' configuration at each missingness level.
- Sum-of-ranks (SOR) aggregated across mis_var_prop=seq(.1,.8,.1) shows quantile- or LOQ-based bounds are not severely worse than the default minimum-value bound.
- Imputed values respect the configured truncation point — left-censored imputations stay below the supplied hi bound (LOQ or 10% quantile).
- Scale and exponential recovery reproduce the original variable distributions after imputation on the scaled log data.
- Output CSV contains the imputed matrix for every tested configuration (default, LOQ, 10% quantile, rsym lower-bound, +/-Inf).

## Limitations

- GSimp with non-informative +/-Inf bounds extends to MCAR/MAR but is primarily designed for left-censored MNAR data caused by LOD/LOQ in targeted MS metabolomics.
- The paper does not prescribe exact quantile thresholds for all conditions; known LOQ/LOD or e.g. 10% quantiles must be chosen per dataset ('for different conditions').
- Small iteration counts (iters_all=10, iters_each=50) may slightly affect accuracy; convergence prefers iters_all=20, iters_each=100.
- Performance in other -omics or complex scenarios beyond targeted metabolomics has not been established and warrants future evaluation.

## Evidence

- [readme] Here, lo=-Inf, hi='min' are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [discussion] known LOQ/LOD or quantile of observed values can be set as upper truncation points for different conditions: "known LOQ/LOD of metabolites or quantile of observed values (e.g., 10%) can be set as upper truncation points for different conditions"
- [readme] quantile values can be applied if minimum is too strict... hi=sapply(data, function(x) quantile(x, .1, na.rm=T)) represent the 10% quantiles of each variable are set to the upper bounds: "notably, quantile values can be applied if minimum is too strict. For example, hi=sapply(data, function(x) quantile(x, .1, na.rm=T)) represent the 10% quantiles"
- [readme] When non-informative bounds for both upper and lower limits were applied, GSimp could be extended to the situation of MCAR/MAR: "When non-informative bounds for both upper and lower limits (e.g., +∞, -∞) were applied, GSimp could be extended to the situation of MCAR/MAR."
- [readme] 'rsym' will draw samples from the left tail of the distribution and symmetrically transformed to the right tail, this is for the right-censored missing: "‘rsym’ will draw samples from the left tail of the distribution and symmetrically transformed to the right tail, this is for the right-censored missing"
