---
name: z-score-standardization-with-back-transformation
description: Decision support for imputing missing metabolite values where missingness
  is due to missing at random combined with a limit of detection (LOD); apply z-score
  standardization before computing neighbor distances and back-transform imputed values
  afterwards.
when_to_use_negative:
- When there is no limit of detection involved — if missingness is due to a metabolite
  truly being absent or purely MAR with no threshold below which values are censored,
  the truncated-normal MLE has no truncation point to anchor and this skill's rationale
  does not apply.
- When metabolite data are strongly non-normal — the method relies on a normality
  assumption for estimating the truncated mean and SD, so heavy skew or multimodal
  distributions violate the model.
- When there are very small samples or excessive missing data per metabolite — estimation
  of the truncated normal mean and variance becomes unstable, giving higher RMSE variability.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: R
  role: 'Implementation language for the KNN-TN imputation pipeline: loading the metabolite
    matrix, fitting truncated-normal MLEs via Newton-Raphson, standardizing, computing
    correlation-based KNN imputation, and back-transforming imputed values.'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:f2cb424a7b19dc7704f11f60e68012faa5eff408e61473b24bea97fa0e6bbd87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# z-score-standardization-with-back-transformation

## Summary

Standardizes metabolites to mean zero and SD one using truncation-based maximum likelihood estimates of the mean and SD, imputes missing values on the standardized scale, then back-transforms imputed values to the original metabolite scale.

## When to use

Decision support for imputing missing metabolite values where missingness is due to missing at random combined with a limit of detection (LOD); apply z-score standardization before computing neighbor distances and back-transform imputed values afterwards.

## When NOT to use

- When there is no limit of detection involved — if missingness is due to a metabolite truly being absent or purely MAR with no threshold below which values are censored, the truncated-normal MLE has no truncation point to anchor and this skill's rationale does not apply.
- When metabolite data are strongly non-normal — the method relies on a normality assumption for estimating the truncated mean and SD, so heavy skew or multimodal distributions violate the model.
- When there are very small samples or excessive missing data per metabolite — estimation of the truncated normal mean and variance becomes unstable, giving higher RMSE variability.

## Inputs

- Metabolite intensity matrix (samples x metabolites) with missing values (e.g. from mass spectrometry metabolomics data)
- Minimum value / limit of detection (LOD) per metabolite, used as the truncation point

## Outputs

- Completed metabolite matrix with imputed values back-transformed to the original metabolite scale (e.g. knn_tn_imputed.csv)

## How to apply

After the cleaning step that removes metabolites with more than 75% missing observations, for each metabolite whose sample mean lies within three standard deviations of the LOD fit a left-truncated normal distribution (truncation point at the LOD) via maximum likelihood using Newton-Raphson optimization initialized at the sample mean and sample SD; otherwise use the sample mean and SD. Standardize every metabolite to mean zero and SD one using these truncation-based estimates rather than sample estimates. On the standardized scale, impute missing values as the weighted average of the K=10 nearest neighbors (correlation-based distance d_C = 1 - |r|, weights w_k = sign(r_k) * d_k^-1 / sum_l(d_l^-1), omitting pairwise missing observations). Finally, back-transform the imputed values to the original metabolite scale using the estimated means and SDs. The rationale is that truncation-based estimates of the mean and SD better reflect the truncated metabolite distribution, improving imputation accuracy compared to using sample estimates.

## Related tools

- **R** (Implementation language for the KNN-TN imputation pipeline: loading the metabolite matrix, fitting truncated-normal MLEs via Newton-Raphson, standardizing, computing correlation-based KNN imputation, and back-transforming imputed values.)

## Examples

```
Rscript knn_tn_imputation.R --input metabolite_matrix.csv --output knn_tn_imputed.csv
```

## Evaluation signals

- On a simulation run where true values are known (values removed from a complete matrix), compute RMSE between imputed and true values and confirm KNN-TN yields lower RMSE than KNN-CR and KNN-EU.
- Confirm that imputed values fall within the valid range for values below the LOD (i.e. imputes values below the LOD more accurately than the Euclidean or correlation-based metrics).
- Verify that imputed values have been back-transformed to the original metabolite scale (e.g. match the expected magnitude/units of the input intensity matrix).
- Check that the standardized matrix (prior to imputation) has metabolites with mean zero and SD one based on the truncation-based estimates.
- Validate concordance of downstream statistical significance testing (MLCI) between the imputed and original data, expecting KNN-TN and KNN-CR roughly equivalent and outperforming zero/mean/minimum imputation.

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal.
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN.
- The KNN-TN likelihood is based solely on observed data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated.
- Downstream analysis evaluation is only preliminary; a more comprehensive study of all potential downstream analyses is still needed.

## Evidence

- [discussion] Log transform metabolites to approximately achieve normality prior to imputation: "we log transform them to approximately achieve normality prior to imputation"
- [methods] Truncated normal distribution with truncation point at the LOD fitted via maximum likelihood: [evidence span withheld — anchor status: not_found]
- [methods] Standardize every metabolite to mean zero and SD one using truncation-based estimates: [evidence span withheld — anchor status: not_found]
- [discussion] KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
- [discussion] Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR. This is directly related to the estimation of the mean and variance for the truncated normal distribution"
