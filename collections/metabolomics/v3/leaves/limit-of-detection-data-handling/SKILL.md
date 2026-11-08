---
name: limit-of-detection-data-handling
description: Use when a metabolomics data matrix contains missing values that arise
  from a known limit of detection (MNAR below-LOD missingness) combined with random
  missingness, and downstream statistical testing requires accurate imputation.
when_to_use_negative:
- The dataset has no limit of detection — the article reports that on the African
  Race data set there was no significant difference between any of the KNN imputation
  algorithms and mean imputation, so simple imputation suffices
- Metabolite distributions are strongly non-normal and cannot be made approximately
  normal by log transformation, since KNN-TN relies on the normality assumption for
  estimating the truncated mean and standard deviation
- Missingness is extremely high or samples are very small — estimation of the truncated
  normal mean and variance is unstable in these settings, giving higher RMSE variability
  than simpler methods
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:06c6b56c4aab8f5f05f05e55ad6d01dc9d190309c45ca18c34534c4e592208c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# limit-of-detection-data-handling

## Summary

Handles metabolomics missing values caused by a limit of detection (LOD) by treating the observed data as a left-truncated normal distribution, estimating truncated means/SDs via maximum likelihood, and imputing with a distribution-based KNN (KNN-TN) instead of substituting minimum/zero/mean values.

## When to use

Use when a metabolomics data matrix contains missing values that arise from a known limit of detection (MNAR below-LOD missingness) combined with random missingness, and downstream statistical testing requires accurate imputation.

## When NOT to use

- The dataset has no limit of detection — the article reports that on the African Race data set there was no significant difference between any of the KNN imputation algorithms and mean imputation, so simple imputation suffices
- Metabolite distributions are strongly non-normal and cannot be made approximately normal by log transformation, since KNN-TN relies on the normality assumption for estimating the truncated mean and standard deviation
- Missingness is extremely high or samples are very small — estimation of the truncated normal mean and variance is unstable in these settings, giving higher RMSE variability than simpler methods

## Inputs

- Metabolomics data matrix (log-transformed) with missing values
- Known limit of detection (LOD) value used as the truncation point a
- Per-metabolite observed values after removing metabolites with >75% missing values

## Outputs

- Truncated-normal MLE estimates of mean and standard deviation per metabolite (e.g. truncated_mle_estimates.csv), with non-convergent metabolites flagged
- Imputed metabolomics data matrix with below-LOD and random missing values filled by KNN-TN
- Imputed matrix suitable for downstream significance testing and metabolite list concordance index (MLCI) evaluation

## How to apply

Log-transform metabolites to approximately achieve normality, and first remove metabolites with >75% missing values during cleaning. Treat the LOD as the truncation point a of a left-truncated normal, and for each eligible metabolite write the log-likelihood l = -n*ln(P(Y in (a,inf)|mu,sigma^2)) - n*ln(sqrt(2*pi*sigma^2)) - sum((y_i-mu)^2)/(2*sigma^2). Fit (mu, sigma^2) by Newton-Raphson maximum likelihood initialized with the observed-data sample mean and SD; per the runtime-acceleration rule, apply the truncated estimation only to metabolites whose sample mean lies within three standard deviations of the LOD, using sample mean/SD for the rest. Standardize with these truncated estimates and impute missing entries using nearest neighbors; flag any metabolite where Newton-Raphson fails to converge.

## Evaluation signals

- Newton-Raphson converges for all eligible metabolites (non-convergence can occur when missingness below the LOD is too high) and estimated means/SDs are finite
- Only metabolites whose sample mean lies within three standard deviations of the LOD receive truncated MLE estimation; the rest use the raw sample mean and SD
- Imputation accuracy: KNN-TN yields lower RMSE than KNN-CR, KNN-EU, and zero/mean/minimum-value (LOD) imputation on MAR+LOD missingness data
- Downstream concordance check: KNN-TN's significance calls are comparable to KNN-CR's and superior to the other four methods on the metabolite list concordance index (MLCI)

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN
- The KNN-TN likelihood is based solely on observed data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated into the full-data likelihood
- Non-convergence of Newton-Raphson can occur when missingness below the LOD is too high
- No software, package, or code availability is stated in the paper despite proposing a new algorithm

## Evidence

- [abstract] KNN-TN generally improves imputation performance under MAR+LOD: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
- [intro] Detection limit as truncation point: "we develop an imputation algorithm based on nearest neighbors that considers MNAR and MAR together based on a truncated distribution, with the detection limit considered as the"
- [discussion] Log transform before imputation: "we log transform them to approximately achieve normality prior to imputation"
- [discussion] 75% missing-value cleaning cutoff: "in the cleaning process (which removes metabolites with >75% MVs)"
- [discussion] Normality assumption limitation: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
- [discussion] Instability with excessive missing data: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR. This is directly related to the estimation of the mean and variance for the truncated normal distribution, which"
