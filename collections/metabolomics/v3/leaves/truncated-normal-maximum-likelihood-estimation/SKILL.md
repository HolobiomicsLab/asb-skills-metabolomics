---
name: truncated-normal-maximum-likelihood-estimation
description: Use when metabolomics data contain missing values believed to arise from
  an LOD (MNAR) combined with MAR missingness, and imputation needs distribution-based
  mean/SD estimates instead of sample estimates for standardization.
when_to_use_negative:
- The metabolite's sample mean lies outside three standard deviations of the LOD —
  the article fits the truncated normal via MLE only for such metabolites and otherwise
  uses the sample mean and SD.
- Missingness is exclusively MNAR with essentially no MAR component, or the data are
  known to be strongly non-normal even after log transformation — the article notes
  the reliance on the normality assumption for estimating the truncated mean and standard
  deviation as a limitation.
- Very small samples or excessive missing data — the article reports that estimation
  of the truncated normal mean and variance is unstable in these settings, giving
  higher RMSE variability for KNN-TN.
edam_operation: http://edamontology.org/operation_2241
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for the KNN-TN pipeline in which truncated-normal
    MLE via Newton-Raphson is embedded, including simulation, imputation, RMSE computation,
    and ANOVA validation scripts.
- name: Newton-Raphson optimization
  role: Numerical optimizer used to maximize the truncated-normal likelihood, initialized
    at the sample mean and sample standard deviation, with the LOD as the truncation
    point.
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:1651ed20edaf9ff54b376f0e8dcecc27a13ea12c68ebd3c6da7e57e13b5e5928
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Newton-Raphson optimization
  - Newton-Raphson optimization for truncated normal MLE
  - R
  edam_operation: http://edamontology.org/operation_2241
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# truncated-normal-maximum-likelihood-estimation

## Summary

A maximum likelihood estimation procedure for the mean and standard deviation of a left-truncated normal distribution, using Newton-Raphson optimization with the limit of detection (LOD) as the truncation point, that supplies distribution-aware standardization parameters for the KNN-TN imputation algorithm in metabolomics data.

## When to use

Use when metabolomics data contain missing values believed to arise from an LOD (MNAR) combined with MAR missingness, and imputation needs distribution-based mean/SD estimates instead of sample estimates for standardization.

## When NOT to use

- The metabolite's sample mean lies outside three standard deviations of the LOD — the article fits the truncated normal via MLE only for such metabolites and otherwise uses the sample mean and SD.
- Missingness is exclusively MNAR with essentially no MAR component, or the data are known to be strongly non-normal even after log transformation — the article notes the reliance on the normality assumption for estimating the truncated mean and standard deviation as a limitation.
- Very small samples or excessive missing data — the article reports that estimation of the truncated normal mean and variance is unstable in these settings, giving higher RMSE variability for KNN-TN.

## Inputs

- Metabolite intensity matrix (samples × metabolites) from mass spectrometry, after removal of metabolites with more than 75% missing values
- Per-metabolite limit of detection (LOD) value serving as the truncation point (e.g., defined as a quantile of the complete data)
- Optional log-transformed intensities, since metabolites are log transformed to approximately achieve normality prior to imputation

## Outputs

- Per-metabolite maximum likelihood estimates of the truncated normal mean and standard deviation
- Standardization parameters (estimated mean, estimated SD) used to scale metabolites for KNN-TN and to back-transform imputed values to the original metabolite scale
- A completed metabolite matrix (e.g., written to knn_tn_imputed.csv) with missing values imputed using these truncation-based estimates

## How to apply

For each metabolite, first compute the sample mean and sample standard deviation, then decide whether the metabolite qualifies for truncated estimation: fit by maximum likelihood only when the sample mean lies within three standard deviations of the LOD (used as the left truncation point); otherwise keep the sample estimates. Optimize the truncated-normal log-likelihood — which is based solely on the observed metabolite data — via Newton-Raphson, initializing at the sample mean and sample SD. The resulting MLE mean and SD are then used, in place of the naive sample estimates, to standardize the metabolite to mean zero and SD one before correlation-distance-based neighbor selection in KNN-TN (K = 10, distance d_C = 1 - |r| with inverse-distance weights weighted by sign(r)), and the imputed values are back-transformed to the original scale with these same estimated parameters. Because the raw metabolite data are typically non-normal, log transform them to approximately achieve normality prior to imputation. Note the trade-off documented in the article: this estimation step is what makes KNN-TN roughly six times slower than KNN-EU on a 50-sample × 400-metabolite dataset (19.95 s vs 1.81 s) but is also what lowers RMSE for below-LOD missing values.

## Related tools

- **R** (Implementation language for the KNN-TN pipeline in which truncated-normal MLE via Newton-Raphson is embedded, including simulation, imputation, RMSE computation, and ANOVA validation scripts.)
- **Newton-Raphson optimization** (Numerical optimizer used to maximize the truncated-normal likelihood, initialized at the sample mean and sample standard deviation, with the LOD as the truncation point.)

## Examples

```
fitdistr(obs[!is.na(obs)], "lognormal"); optim(par=c(sample_mean, sample_sd), fn=tn_loglik, method="BFGS") # LOD as truncation point
```

## Evaluation signals

- Only metabolites whose sample mean is within three standard deviations of the LOD receive truncated-normal MLE estimates; all others retain sample mean/SD — verify this branch was applied per metabolite.
- On simulated data where true values are known (values removed from a complete matrix), RMSE between imputed and true values should be lower for KNN-TN than for KNN-CR and KNN-EU.
- Imputed values below the LOD should be more accurate for KNN-TN than for Euclidean- or correlation-based KNN (e.g., metabolites 1, 3, 4, 7, 8, 12, 13 in the Figure 6 comparison).
- ANOVA on RMSE across the 100-replicate simulations (20x400, 50x400, 100x900; blockwise/AR(1)/mixed correlations; 9%, 15%, 30% missing) should show statistically significant KNN-TN vs KNN-CR and KNN-TN vs KNN-EU contrasts.
- Runtime sanity check: truncated estimation dominates runtime — expect roughly 19.95 s for KNN-TN versus 1.81 s for KNN-EU on a 50-sample × 400-metabolite dataset with 15% missingness.

## Limitations

- The likelihood used is based solely on the observed metabolite data; the full data likelihood would include missing data as well, and the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated.
- Reliance on the normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal (hence the log transform prior to imputation).
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN than or similar to KNN-CR.
- No software, package, or code availability is stated in the paper despite proposing the KNN-TN algorithm that embeds this estimation step.

## Evidence

- [methods] KNN-TN first estimates the means and standard deviations via maximum likelihood estimation under a truncated normal distribution, and uses these estimated values (rather than sample estimates) for standardizing the metabolites.: [evidence span withheld — anchor status: not_found]
- [methods] fit a left-truncated normal distribution with the truncation point at the LOD using maximum likelihood via Newton-Raphson optimization (initialized at the sample mean and sample SD); for all other metabolites use the sample mean and sample SD: [evidence span withheld — anchor status: not_found]
- [discussion] A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation: "the reliance on the normality assumption for estimating the truncated mean and standard deviation"
- [discussion] the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR. This is directly related to the estimation of the mean and variance for the truncated normal distribution: "This is directly related to the estimation of the mean and variance for the truncated normal distribution"
- [discussion] The likelihood used in our KNN-TN method is based solely on the observed metabolite data.: "The likelihood used in our KNN-TN method is based solely on the observed metabolite data"
- [discussion] we log transform them to approximately achieve normality prior to imputation: "we log transform them to approximately achieve normality prior to imputation"
