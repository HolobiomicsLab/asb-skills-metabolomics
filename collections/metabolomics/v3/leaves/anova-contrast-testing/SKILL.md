---
name: anova-contrast-testing
description: Use when comparing RMSE distributions of multiple imputation methods
  (e.g. KNN-TN vs KNN-CR vs KNN-EU) across 100 simulated datasets to test whether
  observed differences are statistically significant.
when_to_use_negative:
- RMSE values come from a single replication or non-numeric metric — ANOVA contrast
  testing on a single observation per cell cannot detect method differences
- The comparison goal is downstream statistical concordance (metabolite list concordance
  index, MLCI) rather than error magnitude — use MLCI-based assessment instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_2269
tools:
- name: Multi-factor ANOVA
  role: statistical modeling of RMSE values to test imputation-method contrasts
- name: R
  role: runtime environment for fitting the ANOVA model and contrasts on simulated
    and real-data RMSE results
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1547-6
    title: Distribution based nearest neighbor imputation for truncated high dimensional
      data with applications to pre-clinical and clinical metabolomics studies
  source_content_hash: sha256:0e802e1c17dbd82fc20ee1a9e91fd613fb65abca918a766967e0b9a67d8a7eb6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Multi-factor ANOVA
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# anova-contrast-testing

## Summary

Multi-factor ANOVA modeling of imputation RMSE values with pre-defined contrasts to statistically test whether KNN-TN achieves lower root mean square error than KNN-CR and KNN-EU.

## When to use

Use when comparing RMSE distributions of multiple imputation methods (e.g. KNN-TN vs KNN-CR vs KNN-EU) across 100 simulated datasets to test whether observed differences are statistically significant.

## When NOT to use

- RMSE values come from a single replication or non-numeric metric — ANOVA contrast testing on a single observation per cell cannot detect method differences
- The comparison goal is downstream statistical concordance (metabolite list concordance index, MLCI) rather than error magnitude — use MLCI-based assessment instead

## Inputs

- RMSE values per replication, imputation method (KNN-TN, KNN-CR, KNN-EU), missingness level, correlation structure, and sample size

## Outputs

- ANOVA test statistics and p-values for pre-defined contrasts (KNN-TN vs KNN-CR, KNN-TN vs KNN-EU)
- reported significance of imputation-method differences across configurations

## How to apply

After generating 100 replicate datasets and computing RMSE between imputed and true values for each method and configuration, fit a multi-factor ANOVA on the RMSE values with main effects for imputation method, percent missing (9%, 15%, 30%), and correlation structure type (blockwise positive rho_w=0.7, AR(1) rho=0.9, blockwise mixed positive/negative). Then run pre-defined contrasts comparing KNN-TN vs KNN-CR and KNN-TN vs KNN-EU. The rationale is that consistently lower RMSE in boxplots is only meaningful if the method-by-method difference is significant; in this study the contrasts confirmed KNN-TN was significantly better than KNN-CR and KNN-EU across all three sample-size cases (20x400, 50x400, 100x900), and in every real-data simulation the mean RMSE for KNN-TN was below KNN-CR, with both more than two standard deviations below KNN-EU at p < 0.05.

## Related tools

- **Multi-factor ANOVA** (statistical modeling of RMSE values to test imputation-method contrasts)
- **R** (runtime environment for fitting the ANOVA model and contrasts on simulated and real-data RMSE results)

## Examples

```
R: summary(aov(RMSE ~ method + missing + corr, data=rmse_simulation_results.csv)); contrasts <- c(KNN_TN=-1,KNN_CR=1,KNN_EU=0); glht(anova_model, linfct=mcp(method=contrasts))
```

## Evaluation signals

- KNN-TN vs KNN-EU contrast must be significant (p < 0.05) in every simulated condition, matching Tables 1-3
- KNN-TN mean RMSE should be below KNN-CR in every case, with both more than two standard deviations below KNN-EU
- ANOVA main effects for method, percent missing, and correlation type should be fitted on all 100 replications per configuration
- Reported results should mirror the paper's pattern: KNN-TN significantly better than the other two KNN methods for all three sample-size cases

## Limitations

- KNN-TN's RMSE variability can be higher than KNN-CR's due to unstable estimation of truncated normal mean and variance with excessive missing data or small samples, which may inflate contrast variance
- ANOVA contrast results reflect the normality assumptions underlying both the imputation model (truncated normal) and the test itself; non-normal metabolite data may bias conclusions
- Findings were derived under specific missingness splits (MNAR:MAR 2:1 and 1:2) and may not generalize to other mechanisms

## Evidence

- [methods] ANOVA modeling of the RMSE values showed statistically significant differences between KNN-TN and the other two KNN methods for all three sample-size cases: [evidence span withheld — anchor status: not_found]
- [results] mean RMSE for KNN-TN was below that for KNN-CR: [evidence span withheld — anchor status: not_found]
- [methods] boxplots of RMSE distributions by method, correlation type, and percent missing: [evidence span withheld — anchor status: not_found]
