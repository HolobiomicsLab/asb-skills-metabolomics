---
name: benjamini-hochberg-pvalue-correction
description: Use when running MetaboDiff diff_test on a metabolite measurement table
  with two or more groups (e.g. MYC-high vs AKT1-high), where raw Student's t-test
  or ANOVA p-values must be adjusted for multiple testing across metabolites.
when_to_use_negative:
- When a pre-computed table of adjusted p-values already exists — re-running diff_test
  would only re-derive the same Benjamini-Hochberg corrections.
- When the analysis goal is network-level exploration of sample traits in the metabolic
  correlation network rather than per-metabolite differential testing between groups.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboDiff
  role: Provides diff_test, which applies the Student's t-test (two groups) or ANOVA
    (>2 groups) and corrects p-values by the Benjamini-Hochberg procedure
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Container object holding the metabolite table; diff_test results (pval, adj_pval,
    dm) are stored in its metadata slot
- name: R
  role: Runtime environment (R version 4.0.2 or higher required) in which MetaboDiff
    and diff_test are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:449466d095fb3ed658ebd4576fa2ab9b939a4e786f2dfbf457dc293af00182f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/andreasmock/MetaboDiff
  tools:
  - MetaboDiff
  - MultiAssayExperiment
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# benjamini-hochberg-pvalue-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Correction of p-values for multiple testing via the Benjamini-Hochberg procedure as implemented inside MetaboDiff's diff_test, producing adj_pval columns alongside raw pval and difference-in-means (dm) values for metabolite comparisons.

## When to use

Use when running MetaboDiff diff_test on a metabolite measurement table with two or more groups (e.g. MYC-high vs AKT1-high), where raw Student's t-test or ANOVA p-values must be adjusted for multiple testing across metabolites.

## When NOT to use

- When a pre-computed table of adjusted p-values already exists — re-running diff_test would only re-derive the same Benjamini-Hochberg corrections.
- When the analysis goal is network-level exploration of sample traits in the metabolic correlation network rather than per-metabolite differential testing between groups.

## Inputs

- MultiAssayExperiment object from MetaboDiff containing the norm_imputed slot (metabolite measurement table after knn imputation at cutoff=0.4 and variance stabilizing normalization)
- Group factor column(s) in colData, e.g. 'group' and 'tumor_groups' with levels such as Control, MYC-high, AKT1-high

## Outputs

- T-test results table in the MAE metadata slot (e.g. 'ttest_tumor_groups_MYC-high_vs_AKT1-high') with pval, adj_pval (Benjamini-Hochberg corrected) and dm columns per metabolite
- Exported CSV such as diff_test_human_tumor_groups.csv listing differential abundance statistics for metabolites like arachidonic acid, docosahexaenoic acid and oleic acid

## How to apply

Run diff_test on a normalized, imputed MultiAssayExperiment (after knn_impute with cutoff=0.4 and variance stabilizing normalization via normalize_met), passing group_factors such as c('group','tumor_groups'). For two-group comparisons MetaboDiff applies the Student's t-test and stores per-metabolite pval, adj_pval and dm (difference in means) in the metadata slot under a label like 'ttest_tumor_groups_MYC-high_vs_AKT1-high'; for more than two groups it applies ANOVA instead. The p-values are corrected for multiple testing by the Benjamini-Hochberg procedure, so interpret significance via adj_pval (e.g. arachidonic acid and docosahexaenoic acid at p<0.05, oleic acid at p<0.01 in the Priolo et al. case study) rather than raw pval alone.

## Related tools

- **MetaboDiff** (Provides diff_test, which applies the Student's t-test (two groups) or ANOVA (>2 groups) and corrects p-values by the Benjamini-Hochberg procedure) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Container object holding the metabolite table; diff_test results (pval, adj_pval, dm) are stored in its metadata slot)
- **R** (Runtime environment (R version 4.0.2 or higher required) in which MetaboDiff and diff_test are executed)

## Examples

```
diff_test(norm_imputed_mae, group_factors=c('group','tumor_groups')); results <- metadata(mae)$ttest_tumor_groups_MYC-high_vs_AKT1-high[,c('pval','adj_pval','dm')]
```

## Evaluation signals

- The saved results table (e.g. diff_test_human_tumor_groups.csv) contains pval, adj_pval and dm columns for every metabolite in the comparison.
- adj_pval values are monotonically consistent with raw pval values (Benjamini-Hochberg order preservation) and adj_pval >= pval for all metabolites.
- In the Priolo et al. case study, arachidonic acid and docosahexaenoic acid reach p<0.05 (*) and oleic acid p<0.01 (**) in the MYC-high vs AKT1-high comparison, matching the case-study significance stars.
- The results slot name follows the expected pattern 'ttest_tumor_groups_MYC-high_vs_AKT1-high' inside the MAE metadata.
- For a >2-group comparison, ANOVA (not t-test) statistics are stored, confirming the correct test path was taken before correction.

## Limitations

- MetaboDiff requires R version 4.0.2 or higher.
- The correction operates on the multiple metabolites tested by diff_test; raw pval and adj_pval must be read together since case-study significance stars may reference uncorrected thresholds.
- Missing values must be handled beforehand (knn_impute with a 40% cutoff recommended); diff_test assumes the norm_imputed data.

## Evidence

- [methods] Student's T-test applied before BH correction: "If two groups are compared the function applies the Student`s T-Test"
- [methods] ANOVA path for >2 groups: "If there are more than two groups an analysis of variance (ANOVA) is applied"
- [methods] BH correction statement: "The p-values are corrected for multiple testing by the Benjamini-Hochberg procedure"
- [methods] knn imputation cutoff prerequisite: "We recommend a cutoff of 40% (i.e. 0.4)"
- [results] Case study task finding: [evidence span withheld — anchor status: not_found]
