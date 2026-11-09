---
name: gibbs-sampler-implementation
description: Use when a targeted LC/MS or GC/MS metabolomics matrix contains left-censored
  missing values attributed to detection limits and requires MNAR-aware imputation
  before downstream univariate or multivariate analysis.
when_to_use_negative:
- When the majority of missing values are MCAR/MAR rather than left-censored MNAR
  from LOD/LOQ — the default lo=-Inf, hi='min' truncation would impose an unjustified
  ceiling; use uninformative bounds (+∞/-∞) only if extending GSimp to MCAR/MAR is
  intentional
- When missing retrieval from raw spectral data is possible — the authors state 'missing
  retrieval from raw spectral data will be most recommended' over imputation
- When the data are not log-transformable (already normalized or contain zeros) without
  violating the pre-processing assumptions (log-transform → QRILC init → scale → impute
  → recover → exponentiate)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: Core R package providing GS_impute (the Gibbs sampler) and pre_processing_GS_wrapper
    (one-step preprocessing + imputation)
  repo: https://github.com/WandeRum/GSimp
- name: glmnet
  role: Elastic net prediction model (lambda=0.01, alpha=0.5) used inside each Gibbs
    iteration to predict missing values from complete variables
- name: imputeLCMD
  role: Provides impute.QRILC used to initialize missing values before the Gibbs sampler
    and as the default 'qrilc' initial method
- name: GSimp_evaluation / Impute_wrapper.R (MVI-evaluation)
  role: Supplies wrapper functions, MNAR generation, and the imputation evaluation
    pipeline used to benchmark GSimp against QRILC, kNN-TN, and HM
  repo: https://github.com/WandeRum/MVI-evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:fc61407ebdfa13785a1dedb46ce69c43173d772896bcb59ff79144a1a12b7c0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - GSimp_evaluation / Impute_wrapper.R (MVI-evaluation)
  - R
  - glmnet
  - imputeLCMD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# gibbs-sampler-implementation

## Summary

Implements GSimp, an iterative Gibbs sampler that imputes left-censored (MNAR, LOD/LOQ-driven) missing values in targeted metabolomics data by embedding an elastic net prediction model within truncated-normal resampling.

## When to use

Use when a targeted LC/MS or GC/MS metabolomics matrix contains left-censored missing values attributed to detection limits and requires MNAR-aware imputation before downstream univariate or multivariate analysis.

## When NOT to use

- When the majority of missing values are MCAR/MAR rather than left-censored MNAR from LOD/LOQ — the default lo=-Inf, hi='min' truncation would impose an unjustified ceiling; use uninformative bounds (+∞/-∞) only if extending GSimp to MCAR/MAR is intentional
- When missing retrieval from raw spectral data is possible — the authors state 'missing retrieval from raw spectral data will be most recommended' over imputation
- When the data are not log-transformable (already normalized or contain zeros) without violating the pre-processing assumptions (log-transform → QRILC init → scale → impute → recover → exponentiate)

## Inputs

- Raw metabolomics data matrix (rows = samples, columns = metabolites; e.g. real_data.csv with FFAs and BAs, or targeted_data.csv / untargeted_data.csv) containing left-censored NA entries
- Optional truncation bounds per variable (known LOQ/LOD values or quantile vectors such as quantile(x, .1, na.rm=T))
- Optional pre-initialized data frame for the `initial` argument (e.g. QRILC-imputed log-scaled data)

## Outputs

- Complete imputed data matrix with no NA entries, returned on the original (exponentiated, un-scaled) measurement scale
- Intermediate log-scaled, QRILC-initialized data frame and stored scale parameters used for scaling recovery

## How to apply

Pre-process by log-transforming the raw data, initializing missing entries with QRILC (imputeLCMD's impute.QRILC), then centralizing and scaling for elastic-net prediction; re-introduce NA at the original missing positions. Call GS_impute with iters_each=100, iters_all=20 (acceptable reduced settings: 50/10), initial='qrilc' or a pre-initialized data frame, lo=-Inf, hi='min' (upper bound = minimum observed value, or a quantile such as the 10% quantile per variable), n_cores for parallelization, and imp_model='glmnet_pred'. Inside each iteration, missing variables are ordered sequentially from least to most missing, an elastic net (glmnet, lambda=0.01, alpha=0.5) predicts candidate values y_hat for each variable's missing entries from the complete variables, sigma is estimated from prediction residuals, and imputed values are drawn from a truncated normal N(y_hat, sigma^2) bounded by lo/hi; the sampler repeats for iters_all x iters_each rounds, with the final iteration's samples retained as the imputed values. Finish by recovering the scaling (scale_recover with method='recover' and the stored parameter data frame) and exponentiating to return to the original measurement scale.

## Related tools

- **GSimp** (Core R package providing GS_impute (the Gibbs sampler) and pre_processing_GS_wrapper (one-step preprocessing + imputation)) — https://github.com/WandeRum/GSimp
- **glmnet** (Elastic net prediction model (lambda=0.01, alpha=0.5) used inside each Gibbs iteration to predict missing values from complete variables)
- **imputeLCMD** (Provides impute.QRILC used to initialize missing values before the Gibbs sampler and as the default 'qrilc' initial method)
- **GSimp_evaluation / Impute_wrapper.R (MVI-evaluation)** (Supplies wrapper functions, MNAR generation, and the imputation evaluation pipeline used to benchmark GSimp against QRILC, kNN-TN, and HM) — https://github.com/WandeRum/MVI-evaluation

## Examples

```
source('GSimp.R'); result <- data_raw_log_sc %>% GS_impute(., iters_each=50, iters_all=10, initial = data_raw_log_qrilc_sc_df, lo=-Inf, hi='min', n_cores=2, imp_model='glmnet_pred')
```

## Evaluation signals

- Output matrix contains no NA entries and matches the original data's dimensions and non-missing values exactly (only previously missing positions changed)
- Imputed values fall within the truncation bounds: for left-censored defaults, all imputations ≤ the minimum observed value of their variable (or the chosen quantile/LOQ)
- With default iters_all=20, iters_each=100, reduced settings (10/50) should not severely degrade accuracy — compare imputation accuracy across iterations on simulated MNAR data
- A linear trend between original and imputed values is preserved (comparable to or better than QRILC/kNN-TN benchmarks in the evaluation pipeline)
- Downstream univariate and multivariate analyses (e.g. PLS-DA, OPLS via ropls) show statistical sensitivity consistent with the paper's evaluation metrics

## Limitations

- Convergence requires many iterations (iters_all=20, iters_each=100 recommended); fewer iterations may impair convergence of MCMC parameters
- For very strict minimum-based truncation, quantile bounds (e.g. 10% quantile) or known LOQ/LOD should be used instead, but these are scenario-dependent and not universally defined
- GSimp's applicability to other -omics or complex scenarios beyond targeted metabolomics is untested; authors state future evaluation is needed
- Parallel version runs Gibbs samplers for multiple missing variables simultaneously, which may trade off exact sequential ordering for speed on large-scale datasets
- No version numbers are pinned for glmnet, vegan, ropls, or imputeLCMD in the paper (README lists glmnet 2.0-13, imputeLCMD 2, etc.)

## Evidence

- [methods] GS_impute arguments and defaults: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [readme] Default iteration counts: [evidence span withheld — anchor status: not_found]
- [readme] Pre-processing wrapper steps: "Log-transformation (for non-normal data); Initialization for missing values (e.g., QRILC); Centralization and scaling (for elastic-net prediction); Imputation using GSimp"
- [discussion] Sequential imputation ordering: "we applied a sequential imputation procedure for missing variables from the least number of missing values to the most"
- [abstract] GSimp performance claim: "The results show that GSimp outperforms other imputation methods in terms of imputation accuracy, observation distribution, univariate and multivariate analyses, and statistical sensitivity."
- [abstract] MNAR rationale: "Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics studies and could be recognized as missing not at"
