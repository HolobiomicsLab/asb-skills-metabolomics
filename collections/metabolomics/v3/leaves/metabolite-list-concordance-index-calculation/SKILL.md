---
name: metabolite-list-concordance-index-calculation
description: Use when you need to evaluate whether an imputation method (e.g., KNN-TN,
  KNN-CR, KNN-EU, zero, minimum, mean) preserves the downstream significant-metabolite
  list from t-tests, by comparing it against the complete-data metabolite list via
  the MLCI.
when_to_use_negative:
- Downstream analysis is not a two-group significance test on metabolite lists — the
  MLCI as defined here measures concordance of statistically significant metabolite
  lists, not prediction accuracy (use RMSE instead for imputation accuracy).
- 'Missing data exceed 75% per metabolite: the paper''s cleaning process removes metabolites
  with >75% missing values before imputation, so such entries would not be part of
  the evaluated list in the first place.'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: Metabolomics WorkBench
  role: Source of the African Race data (Project ID PR000010; Ethiopia and Tanzania
    subsets) used to construct the complete-data gold-standard metabolite lists.
- name: Additional file 3 example script
  role: Provided example code for the imputation-and-MLCI comparison pipeline described
    in the paper.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:ffe09a6c777505cf1d23bd3ebde415f9b92dc168d1dc92547bf67bb695f3320a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Additional file 3 example script
  - Metabolomics WorkBench
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metabolite-list-concordance-index-calculation

## Summary

Computes the metabolite list concordance index (MLCI), a similarity metric comparing the list of statistically significant metabolites identified after imputation to the gold-standard list from the complete data, to quantify how well an imputation method preserves downstream testing conclusions.

## When to use

Use when you need to evaluate whether an imputation method (e.g., KNN-TN, KNN-CR, KNN-EU, zero, minimum, mean) preserves the downstream significant-metabolite list from t-tests, by comparing it against the complete-data metabolite list via the MLCI.

## When NOT to use

- Downstream analysis is not a two-group significance test on metabolite lists — the MLCI as defined here measures concordance of statistically significant metabolite lists, not prediction accuracy (use RMSE instead for imputation accuracy).
- Missing data exceed 75% per metabolite: the paper's cleaning process removes metabolites with >75% missing values before imputation, so such entries would not be part of the evaluated list in the first place.

## Inputs

- Significant-metabolite list from the complete dataset (gold standard)
- Significant-metabolite list from each imputed dataset (e.g., KNN-TN, KNN-CR, KNN-EU, zero, minimum, mean imputation)

## Outputs

- Per-simulation MLCI value comparing imputed-data list to complete-data list
- Averaged MLCI (with SD) per dataset and imputation method over 100 simulations
- ANOVA-contrast test results on mean MLCI differences between imputation methods (cf. Tables S13–S15)

## How to apply

Run the intended downstream test — two-group t-tests on mean log intensity — on both the complete data and each imputed dataset to obtain the statistically significant metabolite lists. Compute the MLCI between the imputed-data list and the gold-standard complete-data list, where higher MLCI indicates the imputation better preserved the metabolite list from the complete data. Repeat over the 100 simulated datasets per dataset/imputation-method combination, averaging MLCI values and SDs, and compare mean MLCIs between methods via pre-defined ANOVA contrasts (as in the paper's Additional file 2, Tables S13–S15). The reference pattern to reproduce: KNN-TN and KNN-CR yield the highest MLCI across the three real datasets, e.g., myocardial infarction KNN-TN MLCI 0.510 (SD 0.097) vs. zero imputation 0.061 (SD 0.054).

## Related tools

- **Metabolomics WorkBench** (Source of the African Race data (Project ID PR000010; Ethiopia and Tanzania subsets) used to construct the complete-data gold-standard metabolite lists.)
- **Additional file 3 example script** (Provided example code for the imputation-and-MLCI comparison pipeline described in the paper.)

## Examples

```
Rscript run_imputation_and_mlci.R --dataset myocardial --missing 0.15 --methods zero,min,mean,knn_eu,knn_cr,knn_tn --sims 100
```

## Evaluation signals

- MLCI values must be in [0,1]; a KNN-TN MLCI of 0.510 (SD 0.097) on the myocardial infarction dataset is the reference reproduction point.
- Averaged MLCI ordering must match Table 4: KNN-TN and KNN-CR highest; KNN-EU, zero, minimum, mean lower.
- ANOVA contrasts must show KNN-TN significantly better than zero, minimum, and KNN-EU imputation in two of the three real datasets.
- On the African Race data, the KNN-TN vs. mean-imputation MLCI difference should be non-significant, consistent with the paper's finding.

## Limitations

- Downstream analysis evaluation was only preliminary in the paper; the authors state a more comprehensive study of all potential downstream analyses is needed to fully determine whether improved imputation accuracy translates into better downstream results.
- KNN-TN's truncated-normal MLE standardization can be unstable with excessive missing data or small samples, giving higher RMSE variability for KNN-TN, which may affect MLCI stability.
- The MLCI does not capture RMSE-level imputation accuracy; the paper evaluates both metrics because they can disagree (e.g., KNN-TN and KNN-EU were roughly equivalent in MLCI but KNN-TN had lower RMSE).

## Evidence

- [abstract] MLCI definition and purpose: "the metabolite list concordance index (MLCI) for influence on downstream statistical testing"
- [methods] Downstream t-test setup: [evidence span withheld — anchor status: not_found]
- [methods] Simulation averaging protocol: [evidence span withheld — anchor status: not_found]
- [results] Reproduction target values: [evidence span withheld — anchor status: not_found]
- [discussion] Method ranking pattern: [evidence span withheld — anchor status: not_found]
- [discussion] Caveat on downstream scope: "a more comprehensive study of all potential downstream analyses is needed to fully determine, whether the improved imputation accuracy of the KNN-TN method translates into better downstream"
