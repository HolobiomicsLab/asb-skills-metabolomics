---
name: newton-raphson-numeric-optimization
description: Use when estimating parameters (e.g. mean and SD) of a truncated normal
  distribution via maximum likelihood on left-truncated metabolomics data, where missing
  values below the LOD require iterative numeric optimization.
when_to_use_negative:
- Metabolite data are heavily non-normal, since the likelihood relies on a normality
  assumption for the truncated distribution (noted as a limitation in the paper)
- Missingness below the LOD is excessive or the sample is very small, making truncated-likelihood
  estimation unstable (higher RMSE variability observed for KNN-TN)
- The full likelihood including the missing-data mechanism is needed — the KNN-TN
  likelihood is based solely on observed data and does not model MNAR/MAR/MCAR explicitly
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3512
tools:
- name: R
  role: Implementation environment for the Newton-Raphson optimizer used to fit the
    truncated-normal maximum-likelihood estimates per metabolite in the KNN-TN imputation
    workflow.
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:d9f6219735ee4594625b2c397abfe904459c2e782cbd14baebbc670544ab0ff5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# newton-raphson-numeric-optimization

## Summary

Newton-Raphson iterative root-finding/maximum-likelihood optimization, as applied in KNN-TN to estimate truncated-normal metabolite means and standard deviations from metabolomics data with missing values below the limit of detection.

## When to use

Use when estimating parameters (e.g. mean and SD) of a truncated normal distribution via maximum likelihood on left-truncated metabolomics data, where missing values below the LOD require iterative numeric optimization.

## When NOT to use

- Metabolite data are heavily non-normal, since the likelihood relies on a normality assumption for the truncated distribution (noted as a limitation in the paper)
- Missingness below the LOD is excessive or the sample is very small, making truncated-likelihood estimation unstable (higher RMSE variability observed for KNN-TN)
- The full likelihood including the missing-data mechanism is needed — the KNN-TN likelihood is based solely on observed data and does not model MNAR/MAR/MCAR explicitly

## Inputs

- Metabolite intensity matrix with missing values (data with values below the limit of detection)
- Limit of detection (LOD) value used as the truncation point a for the left-truncated normal
- Initial parameter guesses: observed-data sample mean and sample standard deviation per metabolite

## Outputs

- Maximum-likelihood estimates of the truncated-normal mean and standard deviation per metabolite (written to truncated_mle_estimates.csv in the KNN-TN workflow)
- Convergence status / flag for metabolites where Newton-Raphson fails to converge

## How to apply

For each metabolite, omit missing values and write the log-likelihood of the left-truncated normal (truncation point a = LOD): l = -n*ln(P(Y in (a,inf)|mu,sigma^2)) - n*ln(sqrt(2*pi*sigma^2)) - sum((y_i-mu)^2)/(2*sigma^2). Initialize the Newton-Raphson optimizer with the observed-data sample mean and sample standard deviation, then iterate NR updates on (mu, sigma^2) until convergence. Apply truncated-normal estimation only to metabolites whose sample mean lies within three standard deviations of the LOD (a runtime-acceleration rule from the paper); for remaining metabolites use the sample mean and SD directly. Flag any metabolite where NR fails to converge — the paper notes this can happen when missingness below the LOD is too high, making the truncated-likelihood estimation unstable.

## Related tools

- **R** (Implementation environment for the Newton-Raphson optimizer used to fit the truncated-normal maximum-likelihood estimates per metabolite in the KNN-TN imputation workflow.)

## Examples

```
Rscript fit_truncated_mle.R --input metabolite_matrix.csv --lod <LOD_value> --output truncated_mle_estimates.csv
```

## Evaluation signals

- Newton-Raphson converges for all eligible metabolites (per the paper, non-convergence can occur when missingness below the LOD is too high)
- Estimated truncated-normal means and standard deviations are finite and usable as standardization parameters for the downstream KNN-TN imputation
- Downstream check: on a simulation where true values are known, compute RMSE between imputed and true values and confirm KNN-TN (which consumes these estimates) yields lower RMSE than KNN-CR and KNN-EU
- Runtime filter applied: truncated-normal MLE was run only for metabolites whose sample mean lies within three standard deviations of the LOD
- Log-likelihood value improves monotonically across NR iterations for each metabolite

## Limitations

- Reliance on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal (paper limitation)
- Estimation of truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN (paper limitation)
- The likelihood used is based solely on observed data; the full-data likelihood including the missingness mechanism (MNAR, MAR, MCAR) is not specified (paper limitation)
- Non-convergence can occur when missingness below the LOD is too high

## Evidence

- [methods] Newton-Raphson used for truncated-normal MLE: [evidence span withheld — anchor status: not_found]
- [methods] Initialization and three-SD runtime rule: [evidence span withheld — anchor status: not_found]
- [discussion] Unstable estimation with excessive missingness: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR. This is directly related to the estimation of the mean and variance for the truncated normal distribution"
- [discussion] Likelihood uses only observed data: "The likelihood used in our KNN-TN method is based solely on the observed metabolite data. The full data likelihood would include missing data as well."
- [discussion] Normality assumption limitation: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
