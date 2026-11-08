---
name: metabolomics-missing-value-imputation
description: Use when a targeted LC/MS metabolomics data matrix contains left-censored
  missing values caused by LOD/LOQ (MNAR) and imputation quality must be evaluated
  before univariate or multivariate analysis such as PLS-DA.
when_to_use_negative:
- Missing values are mostly MCAR/MAR rather than left-censored LOD/LOQ — the authors
  state that non-informative bounds (+Inf/-Inf) would be needed, i.e., GSimp's default
  left-censored setup is the wrong choice
- The dataset is a non-targeted GC/MS profile with heavy missingness before manual
  missing retrieval — the authors recommend missing retrieval from raw spectral data
  first, and note the majority of such missing values are MCAR/MAR
- The task is feature selection or peak annotation rather than filling a missing-value
  matrix in an existing metabolomics sample x variable table
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: GSimp
  role: Core iterative Gibbs sampler imputation function (GS_impute) for left-censored
    missing values, with elastic-net (glmnet_pred) prediction embedded
  repo: https://github.com/WandeRum/GSimp
- name: QRILC
  role: Baseline imputation method used both as initialization inside GSimp and as
    a comparison method (impute.QRILC wrapper)
  repo: https://github.com/WandeRum/GSimp
- name: kNN-TN
  role: Comparison imputation method (truncation kNN, imputeKNN with k=3, distance='truncation')
    from Trunc_KNN
  repo: https://github.com/WandeRum/GSimp
- name: vegan
  role: PCA-Procrustes evaluation (procrustes / Procrustes_cal_plot with DR='PCA',
    nPCs=2) of observation distribution preservation
- name: ropls
  role: PLS-DA evaluation of multivariate analysis distortion after imputation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1005973
    title: 'GSimp: A Gibbs sampler based left-censored missing value imputation approach
      for metabolomics studies'
  source_content_hash: sha256:6154da50c9ead4147d1d3260e170bf0d13f5d2b6b028716b807cdbad34ecbdf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WandeRum/GSimp
  tools:
  - GSimp
  - QRILC
  - R
  - kNN-TN
  - ropls
  - vegan
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 24
---

# metabolomics-missing-value-imputation

## Summary

Iterative Gibbs-sampler based imputation (GSimp) of left-censored (MNAR) missing values in MS-based targeted metabolomics matrices, benchmarked against QRILC and kNN-TN on imputation accuracy, distribution preservation, and downstream statistical sensitivity.

## When to use

Use when a targeted LC/MS metabolomics data matrix contains left-censored missing values caused by LOD/LOQ (MNAR) and imputation quality must be evaluated before univariate or multivariate analysis such as PLS-DA.

## When NOT to use

- Missing values are mostly MCAR/MAR rather than left-censored LOD/LOQ — the authors state that non-informative bounds (+Inf/-Inf) would be needed, i.e., GSimp's default left-censored setup is the wrong choice
- The dataset is a non-targeted GC/MS profile with heavy missingness before manual missing retrieval — the authors recommend missing retrieval from raw spectral data first, and note the majority of such missing values are MCAR/MAR
- The task is feature selection or peak annotation rather than filling a missing-value matrix in an existing metabolomics sample x variable table

## Inputs

- Targeted metabolomics peak/intensity matrix with missing values (CSV, e.g., real_data.csv, targeted_data.csv, untargeted_data.csv)
- Simulated complete data matrix (data_sim.csv, 160 samples x 76 variables) for MNAR-degradation benchmarking
- Missing-position indicator (NA positions in the raw matrix)

## Outputs

- Complete imputed data matrix (log-scale and original-scale after scale/exponential recovery)
- Imputation accuracy metrics per method (RMSE/R2 across mis_var_prop levels)
- PCA-Procrustes sum of squared errors table (pro_ss_df) comparing GSimp, QRILC, and kNN-TN
- Univariate t-test p-values and FDR-adjusted p-values per imputation method
- PLS-DA score and contribution plots per imputation method

## How to apply

Load the metabolomics matrix (e.g., real_data.csv, data_sim.csv, targeted_data.csv, or untargeted_data.csv from WandeRum/GSimp) and pre-process it inside pre_processing_GS_wrapper: log-transformation for non-normal data, QRILC initialization via impute.QRILC, centralization and scaling for the elastic-net prediction model (glmnet_pred), then GS_impute with iters_each=50 and iters_all=10 (or iters_each=100, iters_all=20 for better MCMC convergence), lower bound lo=-Inf and upper bound hi='min' set to the minimum observed value of each variable (quantiles such as the 10% quantile can be used if 'min' is too strict), n_cores=2, followed by scale recovery and exponential recovery. Missing variables are imputed sequentially from least to most missing. Compare GSimp against QRILC (impute.QRILC) and kNN-TN (imputeKNN with k=3, distance='truncation') wrappers, simulating MNAR patterns with MNAR_gen_imp using mis_var_prop=seq(.1,.8,.1) and per-variable missing proportions drawn from U(0.3,0.6). Evaluate accuracy with RMSE/R2, distribution preservation with PCA-Procrustes sum of squared errors on 2 PCs (vegan procrustes via Procrustes_cal_plot with DR='PCA'), univariate t-tests (p-values and FDR-adjusted p-values), and PLS-DA score/contribution plot distortion.

## Related tools

- **GSimp** (Core iterative Gibbs sampler imputation function (GS_impute) for left-censored missing values, with elastic-net (glmnet_pred) prediction embedded) — https://github.com/WandeRum/GSimp
- **QRILC** (Baseline imputation method used both as initialization inside GSimp and as a comparison method (impute.QRILC wrapper)) — https://github.com/WandeRum/GSimp
- **kNN-TN** (Comparison imputation method (truncation kNN, imputeKNN with k=3, distance='truncation') from Trunc_KNN) — https://github.com/WandeRum/GSimp
- **vegan** (PCA-Procrustes evaluation (procrustes / Procrustes_cal_plot with DR='PCA', nPCs=2) of observation distribution preservation)
- **ropls** (PLS-DA evaluation of multivariate analysis distortion after imputation)

## Examples

```
source('Impute_wrapper.R'); data_imp <- pre_processing_GS_wrapper(data_raw)
```

## Evaluation signals

- GSimp should show the lowest Procrustes sum of squared errors between original and imputed PCA ordinations (2 PCs) compared with QRILC and kNN-TN as missing-variable proportion increases
- GSimp should yield favorable linear trend between imputed and original values and imputed values should not exceed the upper truncation bound (hi='min' of observed values)
- RMSE and R2 on the simulation benchmark should favor GSimp over QRILC and kNN-TN across mis_var_prop = 0.1 to 0.8
- Univariate t-test p-values and FDR-adjusted p-values on GSimp-imputed data should be closest to the complete-data ground truth compared with HM, which the authors report severely distorts the data distribution
- PLS-DA score/contribution plots from GSimp-imputed data should best preserve group separation relative to the original complete data

## Limitations

- Default bounds lo=-Inf, hi='min' assume left-censored MNAR; extending to MCAR/MAR requires non-informative bounds (+Inf/-Inf) and is a different regime
- A large number of iterations (iters_all=20, iters_each=100) is recommended for MCMC convergence; smaller settings reduce runtime but the authors note convergence should be checked
- If the minimum observed value is too strict as an upper bound, quantiles (e.g., 10% quantile) should be used instead — choosing the wrong bound distorts the imputation
- Performance in other complex '-omics' scenarios beyond targeted metabolomics has not been evaluated and is left for future work
- Missing values retrieved only after raw spectral data inspection may follow MCAR/MAR rather than MNAR, weakening the left-censored assumption

## Evidence

- [readme] GSimp is a gibbs sampler based left-censored missing value imputation approach for metabolomics studies: "GSimp is a gibbs sampler based left-censored missing value imputation approach for metabolomics studies."
- [readme] data pre-processing steps: "In GSimp, we recommend data pre-processing steps as following:

-   Log-transformation (for non-normal data)
-   Initialization for missing values (e.g., QRILC)
-   Centralization and scaling (for"
- [readme] imputation bounds: "Here, lo=-Inf, hi=‘min’ are default setting for left-censored missing values where the upper bound is set to the minimum value of non-missing part"
- [readme] quantile upper bounds: "quantile values can be applied if minimum is too strict. For example, hi=sapply(data, function(x) quantile(x, .1, na.rm=T)) represent the 10% quantiles"
- [abstract] MNAR cause: "Missing values caused by the limit of detection/quantification (LOD/LOQ) were widely observed in mass spectrometry (MS)-based targeted metabolomics studies and could be recognized as missing not at"
- [discussion] GSimp evaluation scope: "We evaluated GSimp with other three imputation methods (kNN-TN, QRILC, and HM)"
- [discussion] HM distortion: "our results indicated that HM could severely distort the data distribution"
- [discussion] iterative imputation order: "we applied a sequential imputation procedure for missing variables from the least number of missing values to the most"
- [discussion] missing retrieval recommendation: "missing retrieval from raw spectral data will be most recommended"
