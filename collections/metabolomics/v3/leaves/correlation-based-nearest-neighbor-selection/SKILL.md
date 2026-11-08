---
name: correlation-based-nearest-neighbor-selection
description: Use when imputing missing values (MVs) in a metabolite intensity matrix
  where missingness combines MAR and values below a limit of detection (LOD), and
  a correlation-based neighbor set is needed for weighted-average imputation.
when_to_use_negative:
- The dataset shows no missingness due to a threshold/LOD (e.g. purely MAR missingness)
  — on the African Race data set there was no significant difference between any of
  the KNN imputation algorithms and mean imputation
- The matrix has excessive missing data or very small samples (<20), where estimation
  of the truncated normal mean and variance is unstable and gives higher RMSE variability
  for KNN-TN
- Metabolite distributions are strongly non-normal and cannot be normalized by log
  transformation, violating the normality assumption used for the truncated mean and
  SD estimation
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation language for the cleaning step, truncated-normal MLE standardization,
    correlation-based KNN neighbor search, weighted imputation, and writing knn_tn_imputed.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:764758e18ac0a1bb3e85ccf20fd16c4d912a51e3c40ae2224c2bc76fc182c04c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# correlation-based-nearest-neighbor-selection

## Summary

Selects K=10 nearest-neighbor metabolites for missing-value imputation using the correlation-based distance d_C = 1 - |r| on Pearson correlations, with pairwise omission of missing observations, as the neighbor-selection step of the KNN-TN truncated-normal imputation algorithm for metabolomics data.

## When to use

Use when imputing missing values (MVs) in a metabolite intensity matrix where missingness combines MAR and values below a limit of detection (LOD), and a correlation-based neighbor set is needed for weighted-average imputation.

## When NOT to use

- The dataset shows no missingness due to a threshold/LOD (e.g. purely MAR missingness) — on the African Race data set there was no significant difference between any of the KNN imputation algorithms and mean imputation
- The matrix has excessive missing data or very small samples (<20), where estimation of the truncated normal mean and variance is unstable and gives higher RMSE variability for KNN-TN
- Metabolite distributions are strongly non-normal and cannot be normalized by log transformation, violating the normality assumption used for the truncated mean and SD estimation

## Inputs

- Metabolite intensity matrix (log-transformed) with missing values, in CSV form
- Limit of detection (LOD/minimum value) per metabolite used as the truncation point

## Outputs

- Completed metabolite intensity matrix written to knn_tn_imputed.csv with imputed values back-transformed to the original scale
- Optionally, RMSE between imputed and true values on simulation runs where true values are known

## How to apply

First clean the metabolite intensity matrix by removing metabolites with more than 75% missing values, and log-transform to approximately achieve normality prior to imputation. For each metabolite whose sample mean lies within three standard deviations of the LOD, estimate mean and SD by maximum likelihood under a left-truncated normal distribution (truncation point at the LOD, Newton-Raphson initialized at the sample mean/SD); otherwise use sample estimates, then standardize to mean zero and SD one. For each missing value in sample i, find the K=10 nearest neighboring metabolites using the correlation-based distance d_C = 1 - |r| on Pearson correlations, omitting pairwise missing observations during distance calculation. Impute the missing value as the weighted average of the K neighbors with weights w_k = sign(r_k) * d_k^-1 / sum_l(d_l^-1), so that negatively correlated neighbors can also contribute. Back-transform the imputed values to the original metabolite scale using the truncation-based means and SDs and write the completed matrix to knn_tn_imputed.csv.

## Related tools

- **R** (Implementation language for the cleaning step, truncated-normal MLE standardization, correlation-based KNN neighbor search, weighted imputation, and writing knn_tn_imputed.csv)

## Examples

```
Rscript knn_tn_impute.R --input metabolites.csv --lod lod_values.csv --K 10 --output knn_tn_imputed.csv
```

## Evaluation signals

- On a simulation run where true values are known (values removed from a complete matrix), KNN-TN should yield lower RMSE than KNN-CR and KNN-EU
- KNN-TN should impute values below the LOD more accurately than the Euclidean or correlation-based distance metrics
- Imputed values must lie back on the original metabolite scale after reversal of the truncation-based standardization (mean zero, SD one)
- Metabolites retained after cleaning must have at most 75% missing values
- Each imputed value must be a weighted average of exactly K=10 neighbors with weights w_k = sign(r_k) * d_k^-1 normalized to sum to 1

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN
- The likelihood is based solely on observed data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated
- On the African Race data set there was no significant difference between any of the KNN imputation algorithms and mean imputation

## Evidence

- [abstract] abstract comparison: "we show that KNN-TN has lower RMSE values compared to the other two KNN procedures as well as simpler imputation methods based on substituting missing values with the metabolite mean, zero values, or"
- [intro] intro comparison design: "The proposed truncation-based KNN method is compared to standard KNN imputation based on Euclidean and correlation based distance metrics"
- [discussion] cleaning threshold: "in the cleaning process (which removes metabolites with >75% MVs)"
- [discussion] RMSE advantage over simpler methods: "the proposed method (KNN-TN) generally has lower RMSE values compared to the other two KNN methods and simpler imputation algorithms (zero, mean, and minimum value imputation)"
- [discussion] RMSE variability limitation: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR"
