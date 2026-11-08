---
name: metabolomics-missingness-simulation
description: Use when benchmarking metabolite imputation algorithms on real metabolomics
  data where missingness combines MNAR (below limit of detection) and MAR mechanisms,
  and averaged RMSE comparisons across missingness levels are needed.
when_to_use_negative:
- The dataset contains essentially no values below the limit of detection, so missingness
  is purely MAR/MCAR and the truncated-normal KNN-TN machinery adds no benefit.
- The evaluation dataset behaves like the African Race data set, where the paper found
  'no significant difference between any of the KNN imputation algorithms and mean
  imputation'.
- Sample sizes are under 20 or missing percentages fall below those evaluated in the
  study (9%, 15%, 30%) — the paper reports such cases only as 'data not shown'.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: KNN-TN truncation-based imputation
  role: Candidate imputation method being benchmarked; uses correlation distance on
    truncated-normal MLE-standardized data with LOD as truncation point
- name: Newton-Raphson optimization
  role: Estimates the truncated-normal mean and standard deviation for metabolites
    whose sample mean lies within three standard deviations of the LOD
- name: Gene expression distribution simulation approach
  role: Generates the 100 replicate datasets per group with the same missingness mechanism
    as the real data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:c034c17cf3bef5a4bc8e36ef513c9f4de62833f388f23a74560d018a7f1f1116
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gene expression distribution simulation approach
  - KNN-TN truncation-based imputation
  - Newton-Raphson optimization
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# metabolomics-missingness-simulation

## Summary

Reproduces the paper's real-data missingness simulation: 100 replicate datasets generated from three real metabolomics matrices (myocardial infarction, atherothrombotic, African Race) with imposed MNAR+MAR missingness at 9%, 15%, and 30%, to benchmark KNN-TN vs KNN-CR vs KNN-EU imputation by averaged RMSE.

## When to use

Use when benchmarking metabolite imputation algorithms on real metabolomics data where missingness combines MNAR (below limit of detection) and MAR mechanisms, and averaged RMSE comparisons across missingness levels are needed.

## When NOT to use

- The dataset contains essentially no values below the limit of detection, so missingness is purely MAR/MCAR and the truncated-normal KNN-TN machinery adds no benefit.
- The evaluation dataset behaves like the African Race data set, where the paper found 'no significant difference between any of the KNN imputation algorithms and mean imputation'.
- Sample sizes are under 20 or missing percentages fall below those evaluated in the study (9%, 15%, 30%) — the paper reports such cases only as 'data not shown'.

## Inputs

- Real metabolomics data matrix: myocardial infarction data (2 groups x 288 metabolites)
- Real metabolomics data matrix: human atherothrombotic data (sCAD/Type1/Type2 groups x 1032 metabolites)
- Real metabolomics data matrix: African Race data (Ethiopia 25 x 1251 and Tanzania 15 x 2250)
- Limit of detection (LOD) / MNAR percentile specification for each metabolite

## Outputs

- Averaged RMSE table mirroring Tables 1-3: mean and standard deviation of RMSE per method (KNN-TN, KNN-CR, KNN-EU), missingness level, sample size, and group, saved as rmse_simulation_results.csv
- Multi-factor ANOVA contrast results (KNN-TN vs KNN-CR, KNN-TN vs KNN-EU) with p-values

## How to apply

Log-transform each metabolite to approximately achieve normality prior to imputation, then impute any pre-existing missing values with KNN-CR (correlation distance d_C = 1 - |r|). Use the gene-expression distribution simulation approach to generate 100 replicate datasets per group from the three real matrices. Impose total missingness levels of 9%, 15%, and 30% with MNAR (below-LOD values below the MNAR percentile removed) and MAR (randomly removed) in 2:1 and 1:2 combinations, and drop metabolites with more than 75% missing observations. Impute each replicate with KNN-EU (Euclidean distance), KNN-CR (correlation distance on sample-standardized data), and KNN-TN (correlation distance on truncated-normal MLE-standardized data, with mean/sd via Newton-Raphson for metabolites whose sample mean lies within three standard deviations of the LOD), using K = 10 neighbors and sign-weighted averaging with weights w_k = sign(r_k) * d_k^-1 / sum(d_l^-1). Compute RMSE against the true removed values, average over the 100 replications, and run multi-factor ANOVA with pre-defined contrasts (KNN-TN vs KNN-CR, KNN-TN vs KNN-EU) at p < 0.05.

## Related tools

- **KNN-TN truncation-based imputation** (Candidate imputation method being benchmarked; uses correlation distance on truncated-normal MLE-standardized data with LOD as truncation point)
- **Newton-Raphson optimization** (Estimates the truncated-normal mean and standard deviation for metabolites whose sample mean lies within three standard deviations of the LOD)
- **Gene expression distribution simulation approach** (Generates the 100 replicate datasets per group with the same missingness mechanism as the real data)

## Evaluation signals

- KNN-TN achieves the lowest mean RMSE in every simulated condition, matching the pattern reported in Tables 1-3
- KNN-TN vs KNN-EU ANOVA contrasts are significant at p < 0.05
- In every case the mean RMSE for KNN-TN is below that for KNN-CR
- KNN-TN and KNN-CR RMSE means are more than two standard deviations below KNN-EU's, with p < 0.05
- Each reported RMSE cell averages exactly 100 replicate datasets per group and missingness condition

## Limitations

- KNN-TN relies on a normality assumption for estimating the truncated mean and standard deviation, while metabolite data are often non-normal.
- Estimation of the truncated normal mean and variance is unstable with excessive missing data or small samples, giving higher or comparable RMSE variability for KNN-TN relative to KNN-CR.
- The KNN-TN likelihood is based solely on the observed metabolite data; the missingness mechanism (MNAR, MAR, MCAR) is unknown and not incorporated into the likelihood.

## Evidence

- [discussion] we log transform them to approximately achieve normality prior to imputation: "we log transform them to approximately achieve normality prior to imputation"
- [intro] In this work, we develop an imputation algorithm based on nearest neighbors that considers MNAR and MAR together based on a truncated distribution, with the detection limit considered as the: "In this work, we develop an imputation algorithm based on nearest neighbors that considers MNAR and MAR together based on a truncated distribution, with the detection limit considered as the"
- [abstract] KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
- [discussion] in the cleaning process (which removes metabolites with >75% MVs): "in the cleaning process (which removes metabolites with >75% MVs)"
