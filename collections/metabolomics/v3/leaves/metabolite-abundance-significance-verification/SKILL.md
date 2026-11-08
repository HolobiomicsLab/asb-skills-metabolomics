---
name: metabolite-abundance-significance-verification
description: Use when a preprocessed metabolite measurement matrix has two labeled
  groups (e.g. MYC-high vs AKT1-high prostate cancer) and you need per-metabolite
  significance calls (pval, adj_pval, dm) to confirm a published significance pattern.
when_to_use_negative:
- The comparison involves more than two groups and you expect the two-group Student's
  t-test — MetaboDiff switches to ANOVA in that case, so the t-test-based significance
  verification is not applicable
- The metabolite measurements are raw and unprocessed — imputation (knn_impute, cutoff
  0.4) and variance stabilizing normalization must be applied before diff_test results
  are trustworthy
- The goal is pathway-level or network-level interpretation (e.g. metabolic correlation
  network analysis) rather than per-metabolite differential abundance testing
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: MetaboDiff
  role: Provides diff_test for two-group Student's t-testing with Benjamini-Hochberg
    correction, plus knn_impute and normalize_met preprocessing of the MultiAssayExperiment
  repo: http://github.com/andreasmock/MetaboDiff/
- name: R
  role: Runtime environment for MetaboDiff (requires R version 4.0.2 or higher)
  repo: https://rawgit.com/andreasmock/MetaboDiff/master/vignettes/MetaboDiff_tutorial.html
- name: MultiAssayExperiment
  role: Object class created by create_mae that holds the metabolite measurements
    and sample metadata consumed by diff_test
- name: WGCNA
  role: Dependency installed for MetaboDiff's network-related functionality
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:550d7bfee80a1341f0fa80ef3b660671f89473dfb5df65c1e92befe3005391e0
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
  - WGCNA
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolite-abundance-significance-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies which individual metabolites are differentially abundant between two sample groups by running MetaboDiff's diff_test (Student's t-test with Benjamini-Hochberg correction) on a preprocessed MultiAssayExperiment and confirming the resulting pval/adj_pval/dm values against expected significance stars.

## When to use

Use when a preprocessed metabolite measurement matrix has two labeled groups (e.g. MYC-high vs AKT1-high prostate cancer) and you need per-metabolite significance calls (pval, adj_pval, dm) to confirm a published significance pattern.

## When NOT to use

- The comparison involves more than two groups and you expect the two-group Student's t-test — MetaboDiff switches to ANOVA in that case, so the t-test-based significance verification is not applicable
- The metabolite measurements are raw and unprocessed — imputation (knn_impute, cutoff 0.4) and variance stabilizing normalization must be applied before diff_test results are trustworthy
- The goal is pathway-level or network-level interpretation (e.g. metabolic correlation network analysis) rather than per-metabolite differential abundance testing

## Inputs

- MetaboDiff MultiAssayExperiment object (e.g. case1_human from Priolo et al. 2014) with sample metadata in colData
- Group factor labels in colData defining the two-group comparison (e.g. 'group', 'tumor_groups' with 'Control', 'MYC-high', 'AKT1-high')
- List of target metabolites to verify (e.g. arachidonic acid, docosahexaenoic acid, oleic acid)

## Outputs

- Metadata slot entry 'ttest_tumor_groups_MYC-high_vs_AKT1-high' with pval, adj_pval and dm (difference in means) per metabolite
- CSV table of extracted statistics (e.g. diff_test_human_tumor_groups.csv) with significance stars per metabolite

## How to apply

Load the case study MultiAssayExperiment (e.g. case1_human from Priolo et al. 2014) and subset samples to the groups of interest via colData filtering. Impute missing values with knn_impute using the recommended cutoff of 0.4 (40%), then apply variance stabilizing normalization (normalize_met/vsn) so the variance remains nearly constant, and read the norm_imputed slot. Run diff_test with group_factors covering the comparison (e.g. c('group','tumor_groups')); with exactly two groups the function applies the Student's t-test, with more groups it applies ANOVA, and p-values are corrected for multiple testing by the Benjamini-Hochberg procedure. Extract the pval, adj_pval and dm (difference in means) columns for the metabolites of interest from the metadata slot under 'ttest_tumor_groups_MYC-high_vs_AKT1-high' and check that the significance stars match the expected pattern (e.g. arachidonic acid and docosahexaenoic acid at p<0.05, oleic acid at p<0.01) before saving the table.

## Related tools

- **MetaboDiff** (Provides diff_test for two-group Student's t-testing with Benjamini-Hochberg correction, plus knn_impute and normalize_met preprocessing of the MultiAssayExperiment) — http://github.com/andreasmock/MetaboDiff/
- **R** (Runtime environment for MetaboDiff (requires R version 4.0.2 or higher)) — https://rawgit.com/andreasmock/MetaboDiff/master/vignettes/MetaboDiff_tutorial.html
- **MultiAssayExperiment** (Object class created by create_mae that holds the metabolite measurements and sample metadata consumed by diff_test)
- **WGCNA** (Dependency installed for MetaboDiff's network-related functionality)

## Examples

```
diff_test(norm_imputed, group_factors = c('group','tumor_groups'))  # results in metadata slot 'ttest_tumor_groups_MYC-high_vs_AKT1-high'
```

## Evaluation signals

- The metadata slot contains a result entry named 'ttest_tumor_groups_MYC-high_vs_AKT1-high' after diff_test runs
- Arachidonic acid and docosahexaenoic acid show p<0.05 (*) in the MYC-high vs AKT1-high comparison
- Oleic acid shows p<0.01 (**) in the MYC-high vs AKT1-high comparison
- The saved CSV (diff_test_human_tumor_groups.csv) contains pval, adj_pval and dm columns for all queried metabolites
- Missing values were imputed with knn_impute at cutoff=0.4 and normalization used variance stabilizing normalization before testing

## Limitations

- diff_test applies the Student's t-test only when two groups are compared; with more than two groups an ANOVA is applied instead, changing the test semantics
- p-values are corrected for multiple testing by the Benjamini-Hochberg procedure, so adj_pval thresholds may differ from raw pval thresholds
- The MetaboDiff R package requires R version 4.0.2 or higher
- A k-nearest neighbor imputation cutoff of 40% is recommended; metabolites above this missingness cutoff may need different handling

## Evidence

- [methods] Student's t-test for two-group comparisons: "If two groups are compared the function applies the Student`s T-Test"
- [methods] Benjamini-Hochberg correction: "The p-values are corrected for multiple testing by the Benjamini-Hochberg procedure"
- [methods] knn imputation cutoff 0.4: "We recommend a cutoff of 40% (i.e. 0.4)"
- [methods] variance stabilizing normalization rationale: "Variance stabilizing normalization (vsn) is used to ensure that the variance remains nearly constant"
- [readme] MetaboDiff purpose and scope: "The MetaboDiff packages aims to provide a low-level entry to differential metabolomic analysis with R by starting off with the table of metabolite measurements."
