---
name: rmse-imputation-benchmarking
description: Use when comparing imputation algorithms for metabolomics or other high-dimensional
  data with missing values arising from both below-limit-of-detection (MNAR) and random
  technical loss (MAR), and you need a quantitative RMSE ranking.
when_to_use_negative:
- Data are complete with no missing values to impute, so no imputation benchmark is
  meaningful
- The goal is downstream biological interpretation rather than algorithmic performance
  comparison — MLCI or concordance of statistical significance testing may be more
  appropriate than RMSE alone
- Missingness is entirely MCAR with no below-LOD component, making the truncation-based
  KNN-TN advantage unlikely to materialize
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KNN-TN truncation-based imputation
  role: Proposed imputation method under benchmark; uses correlation distance on truncated-normal
    MLE-standardized data with mean/sd estimated via Newton-Raphson for metabolites
    whose sample mean lies within three standard deviations of the LOD
- name: Newton-Raphson optimization
  role: Estimates truncated normal mean and standard deviation for KNN-TN standardization
- name: KNN-CR imputation
  role: Baseline imputation method using correlation distance on sample-standardized
    data; also used to pre-impute real data before simulation
- name: KNN-EU imputation
  role: Baseline imputation method using Euclidean distance; benchmark comparator
- name: Gene expression distribution simulation approach
  role: Generates 100 replicate datasets per group with controlled missingness mechanism
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:cdbf8bd02c634a6aac92d215aa738fa3641884e7b898026f6dc6465b1347a522
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gene expression distribution simulation approach
  - KNN-CR imputation
  - KNN-EU imputation
  - KNN-TN truncation-based imputation
  - Newton-Raphson optimization
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# rmse-imputation-benchmarking

## Summary

Benchmark metabolomics imputation algorithms (KNN-TN, KNN-CR, KNN-EU, zero, mean, minimum-value) by computing root mean square error (RMSE) of imputed entries against known removed values across replicated simulated missingness conditions.

## When to use

Use when comparing imputation algorithms for metabolomics or other high-dimensional data with missing values arising from both below-limit-of-detection (MNAR) and random technical loss (MAR), and you need a quantitative RMSE ranking.

## When NOT to use

- Data are complete with no missing values to impute, so no imputation benchmark is meaningful
- The goal is downstream biological interpretation rather than algorithmic performance comparison — MLCI or concordance of statistical significance testing may be more appropriate than RMSE alone
- Missingness is entirely MCAR with no below-LOD component, making the truncation-based KNN-TN advantage unlikely to materialize

## Inputs

- Log-transformed metabolomics data matrix (e.g., myocardial infarction: 2 groups × 288 metabolites; atherothrombotic: sCAD/Type1/Type2 × 1032 metabolites; African Race: Ethiopia 25×1251 and Tanzania 15×2250) with known true values for entries that will be made missing
- Simulated replicate datasets (100 per group) with known missingness mechanism (MNAR/MAR proportions, total missingness 9%/15%/30%)
- Detection limit (LOD) value used as truncation point for MNAR simulation and KNN-TN standardization

## Outputs

- Averaged RMSE table (mirroring Tables 1–3) with mean and standard deviation per imputation method, missingness level, sample size, and group
- Multi-factor ANOVA contrast results (KNN-TN vs KNN-CR, KNN-TN vs KNN-EU) with p-values
- Metabolite list concordance index (MLCI) values for downstream statistical testing concordance

## How to apply

Take a real metabolomics matrix (e.g., myocardial infarction 2 groups × 288 metabolites, atherothrombotic 1032 metabolites, or African Race 1251/2250 metabolites), log-transform metabolites to approximate normality, and impute pre-existing missing values with KNN-CR (correlation distance d_C = 1 − |r|). Generate 100 simulated replicate datasets per group using the gene-expression distribution simulation approach, imposing total missingness of 9%, 15%, and 30% split between MNAR (values below the MNAR percentile treated as missing) and MAR (randomly removed) in 2:1 and 1:2 combinations, then remove metabolites with >75% missing observations. Impute each replicate with the candidate algorithms — KNN-EU (Euclidean distance), KNN-CR (correlation distance on sample-standardized data), KNN-TN (correlation distance on truncated-normal MLE-standardized data estimated via Newton-Raphson), and simpler methods (zero, metabolite mean, minimum value/LOD) — using K = 10 neighbors and sign-weighted averaging w_k = sign(r_k)·d_k⁻¹/Σd_l⁻¹. Compute RMSE against the true removed values for each replicate and average over the 100 replications, reporting mean and standard deviation per method, missingness level, sample size, and group. Run multi-factor ANOVA with pre-defined contrasts to test KNN-TN vs KNN-CR and KNN-TN vs KNN-EU differences at p < 0.05. Validation: KNN-TN should achieve the lowest mean RMSE in every simulated condition, with KNN-TN vs KNN-EU contrasts significant.

## Related tools

- **KNN-TN truncation-based imputation** (Proposed imputation method under benchmark; uses correlation distance on truncated-normal MLE-standardized data with mean/sd estimated via Newton-Raphson for metabolites whose sample mean lies within three standard deviations of the LOD)
- **Newton-Raphson optimization** (Estimates truncated normal mean and standard deviation for KNN-TN standardization)
- **KNN-CR imputation** (Baseline imputation method using correlation distance on sample-standardized data; also used to pre-impute real data before simulation)
- **KNN-EU imputation** (Baseline imputation method using Euclidean distance; benchmark comparator)
- **Gene expression distribution simulation approach** (Generates 100 replicate datasets per group with controlled missingness mechanism)

## Evaluation signals

- KNN-TN achieves the lowest mean RMSE in every simulated condition, matching the pattern in Tables 1–3
- KNN-TN vs KNN-EU multi-factor ANOVA contrast is significant at p < 0.05, with KNN-TN mean RMSE more than two standard deviations below KNN-EU's
- In every case the mean RMSE for KNN-TN is below that for KNN-CR (KNN-TN and KNN-CR both outperform KNN-EU in real-data simulations)
- Reported RMSE means and standard deviations are averaged over exactly 100 replications per condition
- Metabolites with >75% missing observations are excluded before RMSE computation

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal
- Estimation of truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN
- The KNN-TN likelihood is based solely on observed data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated
- On the African Race data set there was no significant difference between any of the KNN imputation algorithms and mean imputation
- While KNN-TN was significantly better than zero, minimum value, and KNN-EU imputation in two of three data sets, it was no better than KNN-EU imputation in the third

## Evidence

- [results] In the real-data simulation studies (Tables 1-3), KNN-TN and KNN-CR substantially outperformed KNN-EU (RMSE means more than two standard deviations below KNN-EU's, p < 0.05), and in every case the mean RMSE for KNN-TN was below that for KNN-CR.: [evidence span withheld — anchor status: not_found]
- [methods] Missingness mechanism simulation setup: [evidence span withheld — anchor status: not_found]
- [methods] KNN imputation parameterization: [evidence span withheld — anchor status: not_found]
- [discussion] Normality assumption limitation: "A related limitation is the reliance on the normality assumption for estimating the truncated mean and standard deviation"
- [discussion] African Race dataset performance caveat: "on the African Race data set there was no significant difference between any of the KNN imputation algorithms and mean imputation"
- [abstract] Simulated data RMSE advantage of KNN-TN: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
