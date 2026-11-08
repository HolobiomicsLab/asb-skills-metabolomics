---
name: imputation-rmse-evaluation
description: Use when imputation accuracy must be quantified on data where true values
  are known (simulated or masked complete matrices) and missingness combines MNAR
  below-LOD values with MAR dropouts.
when_to_use_negative:
- There is no ground truth (no complete reference matrix) — RMSE between imputed and
  true values cannot be computed and the ANOVA comparison design collapses
- Missingness is purely MAR/MCAR with no limit of detection — the article's gains
  for KNN-TN specifically arise when 'there is missingness due to missing at random
  combined with an LOD'
- The goal is real-data evaluation only (e.g., the African Race data set, where there
  was no significant difference between any of the KNN imputation algorithms and mean
  imputation) without the simulation layer that this skill prescribes
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3172
tools:
- name: R
  role: 'Runtime-implemented simulation and imputation scripts: generates multivariate-normal
    replicate datasets, applies masking, runs the three KNN variants, and writes the
    completed matrix to knn_tn_imputed.csv'
- name: Multi-factor ANOVA
  role: Tests RMSE main effects (method, percent missing, correlation type) and pre-defined
    contrasts KNN-TN vs KNN-CR and KNN-TN vs KNN-EU
- name: Newton-Raphson optimization
  role: Fits the left-truncated normal (truncation point at the LOD) maximum-likelihood
    mean and SD per metabolite, initialized at the sample mean and sample SD, feeding
    the KNN-TN arm of the comparison
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:0bc312252dac379410c4430278dd5a61f36006ee141cd5a2af6fa556613aced2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Multi-factor ANOVA
  - Newton-Raphson optimization
  - R
  - R (runtime-implemented simulation and imputation scripts)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# imputation-rmse-evaluation

## Summary

Simulation-based evaluation of metabolomics missing-value imputation algorithms (KNN-TN vs KNN-CR vs KNN-EU and simple substitutes) by computing RMSE between imputed and known true values, then testing differences with multi-factor ANOVA and MLCI downstream concordance.

## When to use

Use when imputation accuracy must be quantified on data where true values are known (simulated or masked complete matrices) and missingness combines MNAR below-LOD values with MAR dropouts.

## When NOT to use

- There is no ground truth (no complete reference matrix) — RMSE between imputed and true values cannot be computed and the ANOVA comparison design collapses
- Missingness is purely MAR/MCAR with no limit of detection — the article's gains for KNN-TN specifically arise when 'there is missingness due to missing at random combined with an LOD'
- The goal is real-data evaluation only (e.g., the African Race data set, where there was no significant difference between any of the KNN imputation algorithms and mean imputation) without the simulation layer that this skill prescribes

## Inputs

- Complete metabolite intensity matrices (simulated from a multivariate normal with means from Uniform(-5, 5); three correlation structures: blockwise positive rho_w = 0.7 / rho_off = 0.2, AR(1) with rho = 0.9, and blockwise mixed) with values masked to create known-truth test cases
- Missingness specifications: 9%, 15%, 30% missing with 2:1 MNAR-to-MAR split and a quantile-defined LOD
- Imputed matrices from each candidate method (e.g., knn_tn_imputed.csv) plus the corresponding true-value matrix

## Outputs

- Per-replication RMSE table (replication x method x configuration)
- Boxplots of RMSE distributions by method, correlation type, and percent missing for each sample size (mirroring Figures 3-5)
- Multi-factor ANOVA results with pre-defined contrasts (KNN-TN vs KNN-CR, KNN-TN vs KNN-EU) and p-values
- Metabolite list concordance index (MLCI) values per method for downstream statistical-testing concordance

## How to apply

Generate replicate complete matrices (e.g., 100 replications at 20x400, 50x400, 100x900) from an M-dimensional multivariate normal, then mask 9%, 15%, and 30% of values with a 2:1 MNAR-to-MAR split, where MNAR values are those below the corresponding quantile LOD. Apply the cleaning step removing metabolites with more than 75% missing observations. Impute each replicate with the competing methods — KNN-EU (Euclidean distance), KNN-CR (Pearson correlation distance d_C = 1 - |r| with sample mean/SD standardization), and KNN-TN (correlation distance with mean/SD estimated by truncated-normal MLE via Newton-Raphson, LOD as truncation point) — using K = 10 neighbors and inverse-distance weights with sign(r) to include negatively correlated neighbors. Compute the RMSE between imputed and true values for each replication, method, and configuration, and back-transform imputed values using the estimated means and SDs. Fit a multi-factor ANOVA on RMSE with main effects for method, percent missing, and correlation type, and test pre-defined contrasts (KNN-TN vs KNN-CR, KNN-TN vs KNN-EU) at p < 0.05. Produce boxplots of RMSE distributions by method, correlation type, and percent missing for each sample size, mirroring Figures 3-5, and optionally assess downstream influence with the metabolite list concordance index (MLCI).

## Related tools

- **R** (Runtime-implemented simulation and imputation scripts: generates multivariate-normal replicate datasets, applies masking, runs the three KNN variants, and writes the completed matrix to knn_tn_imputed.csv)
- **Multi-factor ANOVA** (Tests RMSE main effects (method, percent missing, correlation type) and pre-defined contrasts KNN-TN vs KNN-CR and KNN-TN vs KNN-EU)
- **Newton-Raphson optimization** (Fits the left-truncated normal (truncation point at the LOD) maximum-likelihood mean and SD per metabolite, initialized at the sample mean and sample SD, feeding the KNN-TN arm of the comparison)

## Examples

```
rmse <- sqrt(mean((imputed[is.na(orig)] - truth[is.na(orig)])^2))  # RMSE of imputed vs true values for each replication, method, and configuration
```

## Evaluation signals

- KNN-TN shows consistently lower mean RMSE than KNN-CR and KNN-EU across all sample-size, correlation, and percent-missing configurations
- The KNN-TN vs KNN-CR and KNN-TN vs KNN-EU ANOVA contrasts are statistically significant (p < 0.05) for all three sample-size cases
- KNN-TN imputes values below the LOD more accurately than the Euclidean or correlation-based distance metrics
- MLCI values between KNN-TN and KNN-EU are roughly equivalent and superior to the other four methods in most cases
- RMSE distributions reproduce the ordering seen in Figures 3-5 boxplots by method, correlation type, and percent missing for each sample size

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal (the article recommends log transforming to approximately achieve normality prior to imputation)
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN than KNN-CR
- The KNN-TN likelihood is based solely on observed metabolite data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated
- Downstream analysis evaluation via MLCI is only preliminary; a comprehensive study of all potential downstream analyses is still needed
- On the African Race data set there was no significant difference between any of the KNN imputation algorithms and mean imputation

## Evidence

- [abstract] we show that KNN-TN has lower RMSE values compared to the other two KNN procedures as well as simpler imputation methods: "we show that KNN-TN has lower RMSE values compared to the other two KNN procedures as well as simpler imputation methods"
- [abstract] MLCI definition for downstream testing: "the metabolite list concordance index (MLCI) for influence on downstream statistical testing"
- [intro] simulation-based effectiveness claim: "We show that this method is effective and generally outperforms the other two KNN procedures through extensive simulation studies"
- [discussion] discussion confirmation of RMSE ordering: "the proposed method (KNN-TN) generally has lower RMSE values compared to the other two KNN methods and simpler imputation algorithms (zero, mean, and minimum value imputation)"
- [discussion] RMSE variability caveat for KNN-TN: "the variability of the RMSE for KNN-TN is higher than or similar to that for KNN-CR"
