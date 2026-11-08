---
name: log-likelihood-implementation
description: Use when a metabolomics matrix has missing values below a known limit
  of detection (LOD), so observed values are left-truncated, and maximum-likelihood
  (mu, sigma^2) estimates are needed as standardization parameters for KNN-TN imputation.
when_to_use_negative:
- Metabolites whose sample mean lies farther than three standard deviations from the
  LOD — per the paper's runtime rule, use the sample mean and SD directly instead
  of the truncated-normal MLE.
- Data with no limit-of-detection truncation (purely MAR missingness) — the truncated-normal
  likelihood is unnecessary; standard KNN-EU/KNN-CR imputation applies.
- Metabolites with excessive missingness below the LOD or very small samples, where
  the truncated mean/variance estimation is unstable and Newton-Raphson may fail to
  converge.
edam_topics:
- http://edamontology.org/topic_0091
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:6f56bbeb2985f1f12a13be728d3feefa5802bedaf37da37088f49500bcb15b32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# log-likelihood-implementation

## Summary

Implements the KNN-TN paper's left-truncated-normal maximum-likelihood estimation of each metabolite's mean and standard deviation, solved with a Newton-Raphson optimizer so that missing values below the limit of detection are correctly accounted for before KNN-TN standardization.

## When to use

Use when a metabolomics matrix has missing values below a known limit of detection (LOD), so observed values are left-truncated, and maximum-likelihood (mu, sigma^2) estimates are needed as standardization parameters for KNN-TN imputation.

## When NOT to use

- Metabolites whose sample mean lies farther than three standard deviations from the LOD — per the paper's runtime rule, use the sample mean and SD directly instead of the truncated-normal MLE.
- Data with no limit-of-detection truncation (purely MAR missingness) — the truncated-normal likelihood is unnecessary; standard KNN-EU/KNN-CR imputation applies.
- Metabolites with excessive missingness below the LOD or very small samples, where the truncated mean/variance estimation is unstable and Newton-Raphson may fail to converge.

## Inputs

- log-transformed metabolomics data matrix with missing values (metabolites should be log-transformed to approximately achieve normality prior to imputation)
- per-metabolite limit of detection (LOD) values used as the left-truncation point a
- per-metabolite observed-data sample mean and sample standard deviation (used to initialize Newton-Raphson)

## Outputs

- truncated_mle_estimates.csv: estimated truncated mean and standard deviation for every metabolite
- convergence flags for any metabolite where Newton-Raphson fails to converge

## How to apply

For each metabolite, omit missing values and write the log-likelihood of the left-truncated normal with the LOD as truncation point a: l = -n*ln(P(Y in (a,inf)|mu,sigma^2)) - n*ln(sqrt(2*pi*sigma^2)) - sum((y_i-mu)^2)/(2*sigma^2). Initialize the Newton-Raphson optimizer with the observed-data sample mean and sample standard deviation, then iterate NR updates on (mu, sigma^2) until convergence. Apply the truncated-normal estimation only to metabolites whose sample mean lies within three standard deviations of the LOD (the paper's runtime-acceleration rule); for the remaining metabolites use the sample mean and SD directly. The rationale is that values missing below the LOD make the observed data a left-truncated normal, so raw sample moments underestimate the true location and scale. Flag any metabolite where NR fails to converge, which the paper notes can occur when missingness below the LOD is too high.

## Examples

```
theta = newton_raphson(loglik_truncnorm, x0=[np.mean(y_obs), np.std(y_obs)], args=(y_obs, lod_a)); df[['trunc_mean','trunc_sd']].to_csv('truncated_mle_estimates.csv')
```

## Evaluation signals

- Newton-Raphson converges for all eligible metabolites (per the paper, non-convergence can occur when missingness below the LOD is too high); failed metabolites are flagged.
- All estimated truncated means and standard deviations are finite and usable as the standardization parameters for KNN-TN.
- Every metabolite passing the eligibility rule has |sample mean - LOD| within three standard deviations; metabolites failing this rule carry sample mean/SD instead of MLE values.
- Estimates initialized at the observed-data sample mean and sample SD, confirming the optimizer start values match the paper's specification.
- truncated_mle_estimates.csv contains exactly one estimated (mean, SD) pair per metabolite in the input matrix.

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal.
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN.
- The likelihood used is based solely on the observed metabolite data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated into the full-data likelihood.
- Newton-Raphson may fail to converge when missingness below the LOD is too high; such metabolites must be flagged.

## Evidence

- [methods] NR initialization and iteration: [evidence span withheld — anchor status: not_found]
- [methods] Three-SD-of-LOD eligibility rule: [evidence span withheld — anchor status: not_found]
- [discussion] Normality pre-transform: "we log transform them to approximately achieve normality prior to imputation"
- [abstract] Performance motivation: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU"
