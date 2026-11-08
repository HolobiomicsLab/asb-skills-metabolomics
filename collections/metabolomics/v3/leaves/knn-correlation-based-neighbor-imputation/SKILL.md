---
name: knn-correlation-based-neighbor-imputation
description: Use when metabolomics datasets contain missing values (from LOD/MNAR
  or MAR mechanisms) and you need a correlation-distance KNN baseline imputation to
  benchmark against, or to reproduce KNN-CR arms in the paper's RMSE and MLCI evaluation
  workflows.
when_to_use_negative:
- When the goal is the paper's best-performing truncated-normal method, KNN-TN — KNN-CR
  is a comparator arm, and the paper reports KNN-TN generally has lower RMSE than
  KNN-CR
- When missingness is purely below-detection-limit MNAR without consideration of the
  LOD distribution — the paper's recommended method for MAR combined with an LOD is
  KNN-TN, not KNN-CR
- When metabolites have more than 75% missing values and should be removed in the
  cleaning process instead of imputed
edam_operation: http://edamontology.org/operation_3552
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: Metabolomics WorkBench
  role: Source of the African Race data (Project ID PR000010) on which the KNN-CR
    imputation arm and downstream MLCI comparison are run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:1609ab39725bc039e656d72d21b3241610a690d54d8504c4f0f1213d72a06b60
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Additional file 3 example script
  - Metabolomics WorkBench
  edam_operation: http://edamontology.org/operation_3552
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# knn-correlation-based-neighbor-imputation

## Summary

KNN-CR is a nearest-neighbor imputation method for metabolomics missing values in which candidate neighbors are selected using a correlation-based distance on log-intensity data standardized to mean zero and SD one, serving as the primary comparator against the paper's proposed KNN-TN approach.

## When to use

Use when metabolomics datasets contain missing values (from LOD/MNAR or MAR mechanisms) and you need a correlation-distance KNN baseline imputation to benchmark against, or to reproduce KNN-CR arms in the paper's RMSE and MLCI evaluation workflows.

## When NOT to use

- When the goal is the paper's best-performing truncated-normal method, KNN-TN — KNN-CR is a comparator arm, and the paper reports KNN-TN generally has lower RMSE than KNN-CR
- When missingness is purely below-detection-limit MNAR without consideration of the LOD distribution — the paper's recommended method for MAR combined with an LOD is KNN-TN, not KNN-CR
- When metabolites have more than 75% missing values and should be removed in the cleaning process instead of imputed

## Inputs

- Log-transformed metabolomics intensity matrix with missing values (e.g., myocardial infarction data with 288 metabolites, atherothrombotic baseline data with 1032 metabolites, or African Race Ethiopia/Tanzania complete subsets)
- Simulated metabolomics datasets with 15% missingness (10% MNAR below the LOD at the 10% quantile plus 5% MAR)
- K value (K=10) and the correlation-based distance definition (d_C = 1 - |r|)

## Outputs

- Completed metabolomics data matrix with missing entries replaced by correlation-weighted neighbor averages
- RMSE values against the complete data for simulation studies
- Metabolite list concordance index (MLCI) values (mean and SD over 100 simulations) comparing significant-metabolite lists to the complete-data gold standard
- Two-group t-test results on mean log intensity for the imputed data

## How to apply

Start from log-transformed metabolite intensity data (metabolites are log transformed to approximately achieve normality prior to imputation). For each metabolite with missing values, standardize metabolites to mean zero and standard deviation one, then compute the correlation-based distance between metabolites so the K nearest neighbors (K=10 in the paper's pipelines) are those with the most similar abundance profiles across samples. Fill each missing entry with the average of the neighbors' observed values, and repeat the procedure as one arm of a six-method comparison (zero, minimum/LOD, mean, KNN-EU, KNN-CR, KNN-TN) on simulated or real datasets with ~15% missingness (10% MNAR below the 10%-quantile LOD plus 5% MAR). After imputation, remove metabolites with more than 75% missing observations, run two-group t-tests on mean log intensity, and score the imputation by RMSE against complete data or by the metabolite list concordance index (MLCI) against the gold-standard significant-metabolite list.

## Related tools

- **Metabolomics WorkBench** (Source of the African Race data (Project ID PR000010) on which the KNN-CR imputation arm and downstream MLCI comparison are run)

## Examples

```
python knn_impute.py --method KNN-CR --k 10 --dist correlation --input metabolites.csv --output imputed.csv
```

## Evaluation signals

- KNN-CR must produce one of the top MLCI values across the three real datasets — Table 4 pattern shows KNN-TN and KNN-CR have the highest MLCI values (e.g., myocardial KNN-TN 0.510 with SD 0.097; KNN-CR near it), while zero imputation drops to 0.061 (0.054)
- In RMSE simulations, KNN-CR should be beaten by KNN-TN; if KNN-CR RMSE is lowest, the missingness mechanism or K parameter is likely mis-specified
- Metabolites with more than 75% missing observations must be removed before or as part of the imputation pipeline; failure to do so invalidates downstream MLCI
- Mean MLCI and SD over 100 simulations per dataset/method must be reported, and ANOVA contrasts (cf. Tables S13-S15) should show KNN-TN significantly better than zero, minimum, and KNN-EU in two of three datasets, with KNN-CR not significantly worse
- Correlation-based neighbor selection requires standardizing metabolites to mean zero and SD one; skipping this standardization changes which metabolites become neighbors and shifts imputation accuracy

## Limitations

- KNN-CR is generally outperformed by the paper's proposed KNN-TN in RMSE under missingness due to MAR combined with a LOD threshold
- In concordance of statistical significance testing, KNN-CR is only roughly equivalent to KNN-TN and outperforms the other four methods, so it is not the definitive best imputation choice
- KNN-CR was no better than mean imputation on the African Race data set, so its advantage depends on the dataset and missingness structure
- KNN imputation with correlation distance still requires enough fully observed metabolites for stable neighbor correlation estimates, which is strained with excessive missing data or small samples

## Evidence

- [methods] KNN-CR (K=10, correlation-based neighbor selection with metabolites standardized to mean zero and SD one): [evidence span withheld — anchor status: not_found]
- [abstract] KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an: "KNN-TN generally has improved performance in imputing the missing values of the different datasets compared to KNN-CR and KNN-EU when there is missingness due to missing at random combined with an"
- [abstract] the metabolite list concordance index (MLCI) for influence on downstream statistical testing: "the metabolite list concordance index (MLCI) for influence on downstream statistical testing"
- [discussion] in the cleaning process (which removes metabolites with >75% MVs): "in the cleaning process (which removes metabolites with >75% MVs)"
