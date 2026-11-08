---
name: t-test-multiple-group-comparison
description: Use when comparing metabolite abundance between pre-defined sample groups
  (e.g. tumor subtypes like MYC-high vs AKT1-high prostate cancer) in a MultiAssayExperiment
  and you need per-metabolite p-values, adjusted p-values and difference of means.
when_to_use_negative:
- When the input is a raw metabolite measurement table that has not been imputed and
  variance-stabilizing normalized — apply knn_impute (cutoff 0.4) and normalize_met
  first
- When the goal is pathway-level or network-level analysis rather than per-metabolite
  group comparison — use MetaboDiff's metabolic correlation network functionality
  instead
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MetaboDiff
  role: Provides diff_test, knn_impute and normalize_met for the two-/multi-group
    comparison with BH correction
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Container object created by create_mae on which diff_test is run
- name: R
  role: Runtime environment for MetaboDiff (requires R version 4.0.2 or higher)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:bdca955c2d8c47ab4dd71e4537f20dea9cd58bc303d8f66143e30d85cb181c60
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
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# t-test-multiple-group-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies MetaboDiff's diff_test to metabolite measurement tables, running Student's t-test for two-group comparisons or ANOVA for more than two groups, with Benjamini-Hochberg multiple-testing correction, to identify differentially abundant metabolites.

## When to use

Use when comparing metabolite abundance between pre-defined sample groups (e.g. tumor subtypes like MYC-high vs AKT1-high prostate cancer) in a MultiAssayExperiment and you need per-metabolite p-values, adjusted p-values and difference of means.

## When NOT to use

- When the input is a raw metabolite measurement table that has not been imputed and variance-stabilizing normalized — apply knn_impute (cutoff 0.4) and normalize_met first
- When the goal is pathway-level or network-level analysis rather than per-metabolite group comparison — use MetaboDiff's metabolic correlation network functionality instead

## Inputs

- MetaboDiff MultiAssayExperiment object (from create_mae) with a norm_imputed slot after knn_impute (cutoff=0.4) and normalize_met
- Group factor column(s) in colData, e.g. 'tumor_groups' with levels such as Control, MYC-high, AKT1-high

## Outputs

- Per-metabolite test results (pval, adj_pval, dm) stored in the metadata slot, e.g. 'ttest_tumor_groups_MYC-high_vs_AKT1-high'
- Exported table of differential abundance results (e.g. diff_test_human_tumor_groups.csv)

## How to apply

First preprocess the metabolite table: impute missing values with knn_impute (recommended cutoff of 40%, i.e. 0.4), then apply variance stabilizing normalization (vsn) via normalize_met so the variance remains nearly constant. Run diff_test on the resulting MultiAssayExperiment, passing group_factors such as c('group','tumor_groups'); the function decides the test by group count — Student's t-test if two groups are compared, ANOVA if there are more than two — and corrects p-values with the Benjamini-Hochberg procedure. Results are stored in the metadata slot under a name like 'ttest_tumor_groups_MYC-high_vs_AKT1-high'; extract the pval, adj_pval and dm (difference in means) columns per metabolite. Validate by checking significance stars reproduce expectations, e.g. in the Priolo et al. case study arachidonic acid and docosahexaenoic acid at p<0.05 (*) and oleic acid at p<0.01 (**).

## Related tools

- **MetaboDiff** (Provides diff_test, knn_impute and normalize_met for the two-/multi-group comparison with BH correction) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Container object created by create_mae on which diff_test is run)
- **R** (Runtime environment for MetaboDiff (requires R version 4.0.2 or higher))

## Examples

```
diff_test(norm_imputed_mae, group_factors = c('group','tumor_groups'))
```

## Evaluation signals

- Saved results table contains pval, adj_pval and dm columns for each metabolite in the specified group comparison
- In the Priolo et al. case study, arachidonic acid and docosahexaenoic acid reach p<0.05 (*) and oleic acid p<0.01 (**) in the MYC-high vs AKT1-high comparison
- Results object is stored in the metadata slot under 'ttest_tumor_groups_MYC-high_vs_AKT1-high'
- Adjusted p-values reflect Benjamini-Hochberg multiple-testing correction
- Two-group comparisons were handled as Student's t-test; >2 groups would trigger ANOVA

## Limitations

- The knn imputation cutoff should be respected — MetaboDiff recommends a cutoff of 40% (0.4); metabolites missing above that cutoff are not suitable for imputation
- MetaboDiff provides a low-level entry to differential metabolomic analysis; it starts from a table of metabolite measurements and does not handle raw mass-spectrometry processing
- Test type is dictated by group count (t-test for two groups, ANOVA for more), so custom statistical models are not supported via diff_test

## Evidence
