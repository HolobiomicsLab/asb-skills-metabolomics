---
name: t-test-differential-metabolite-detection
description: Use when two-group metabolomics intensity data (e.g. case vs control,
  imputed or complete) must be reduced to a significant-metabolite list, and that
  list's stability across imputation methods must be quantified via MLCI.
when_to_use_negative:
- The comparison involves more than two groups or a continuous outcome — the paper's
  downstream evaluation is specifically a two-group t-test on mean log intensity
- The goal is raw imputation accuracy (RMSE of recovered values) rather than concordance
  of the downstream significance list — the article evaluates these separately
- Metabolites with >75% missing values have not yet been removed; the paper's cleaning
  process excludes them before the MLCI assessment
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Metabolomics WorkBench
  role: Source of the African Race real dataset (Project ID PR000010; Ethiopia 25x1251
    and Tanzania 15x2250 complete subsets) on which the t-test/MLCI evaluation is
    run
- name: Additional file 3 example script
  role: Supplementary script accompanying the paper that documents the simulation,
    t-testing, and MLCI computation pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:a4b9758a877246b32f24072816dec5c89e53a3c310d1c0c57ebfb077c5280370
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Additional file 3 example script
  - Metabolomics WorkBench
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# t-test-differential-metabolite-detection

## Summary

Runs two-group t-tests on mean log metabolite intensity to derive the statistically significant metabolite list, then scores that list against a complete-data gold standard using the metabolite list concordance index (MLCI). This is the downstream-analysis yardstick the paper uses to judge whether an imputation method (e.g. KNN-TN vs zero/minimum/mean/KNN-EU/KNN-CR) preserves statistically valid biological conclusions.

## When to use

Use when two-group metabolomics intensity data (e.g. case vs control, imputed or complete) must be reduced to a significant-metabolite list, and that list's stability across imputation methods must be quantified via MLCI.

## When NOT to use

- The comparison involves more than two groups or a continuous outcome — the paper's downstream evaluation is specifically a two-group t-test on mean log intensity
- The goal is raw imputation accuracy (RMSE of recovered values) rather than concordance of the downstream significance list — the article evaluates these separately
- Metabolites with >75% missing values have not yet been removed; the paper's cleaning process excludes them before the MLCI assessment

## Inputs

- Metabolite intensity matrix (metabolites x samples) after imputation or from complete data, log-transformed
- Two-group sample labels defining the comparison (e.g. case/control or Ethiopia/Tanzania in the African Race data, PR000010)
- Gold-standard significant metabolite list derived from the complete data
- Simulated or real datasets: myocardial infarction (288 metabolites), atherothrombotic (1032 metabolites, baseline), African Race complete subsets (Ethiopia 25x1251, Tanzania 15x2250)

## Outputs

- Statistically significant metabolite list per (imputed) dataset from two-group t-tests on mean log intensity
- MLCI value per dataset per imputation method, with mean and SD averaged over 100 simulations
- ANOVA contrast results comparing mean MLCI of KNN-TN against zero, minimum, mean, KNN-EU, and KNN-CR imputation

## How to apply

First log transform the metabolite intensities to approximately achieve normality prior to testing. For each (imputed) dataset, run two-group t-tests on mean log intensity to obtain the statistically significant metabolite list. Compute the MLCI of that list against the gold-standard significant list obtained from the complete (unmasked) data. In the paper's simulation design this is repeated over 100 simulations per dataset per imputation method (15% missingness: 10% MNAR below the LOD at the 10% complete-data quantile plus 5% MAR), after removing metabolites with more than 75% missing observations. Average MLCI values (report mean and SD), then test mean MLCI differences between KNN-TN and each competing method via pre-defined ANOVA contrasts (Additional file 2: Tables S13-S15).

## Related tools

- **Metabolomics WorkBench** (Source of the African Race real dataset (Project ID PR000010; Ethiopia 25x1251 and Tanzania 15x2250 complete subsets) on which the t-test/MLCI evaluation is run)
- **Additional file 3 example script** (Supplementary script accompanying the paper that documents the simulation, t-testing, and MLCI computation pipeline)

## Evaluation signals

- Reproduce the Table 4 pattern: myocardial KNN-TN MLCI of 0.510 (0.097) and zero imputation MLCI of 0.061 (0.054)
- KNN-TN and KNN-CR should show the highest MLCI values across the three real datasets, with KNN-EU, Zero, Minimum and Mean lower
- MLCI must lie in [0, 1] and be reported as mean with SD over 100 simulations per dataset/method
- ANOVA contrasts must show KNN-TN significantly better than zero, minimum, and KNN-EU imputation in two of three datasets, and no significant difference from mean imputation on the African Race data
- The gold-standard list must come from the complete data and be computed with the identical two-group t-test procedure applied to each imputed dataset

## Limitations

- On the African Race data set there was no significant difference between any of the KNN imputation algorithms and mean imputation, so the t-test/MLCI evaluation may not discriminate methods there
- Downstream analysis evaluation is only preliminary; a more comprehensive study of all potential downstream analyses is needed to confirm whether improved imputation accuracy translates into better downstream results
- KNN-TN's upstream imputation relies on a normality assumption for the truncated distribution, which affects the t-test inputs when metabolite data are non-normal

## Evidence

- [abstract] the metabolite list concordance index (MLCI) for influence on downstream statistical testing: "the metabolite list concordance index (MLCI) for influence on downstream statistical testing"
- [methods] run two-group t-tests on mean log intensity to obtain the statistically significant metabolite list: [evidence span withheld — anchor status: not_found]
- [discussion] Assessment based on concordance in statistical significance testing demonstrate that KNN-TN and KNN-CR are roughly equivalent: "Assessment based on concordance in statistical significance testing demonstrate that KNN-TN and KNN-CR are roughly equivalent"
- [discussion] we log transform them to approximately achieve normality prior to imputation: "we log transform them to approximately achieve normality prior to imputation"
