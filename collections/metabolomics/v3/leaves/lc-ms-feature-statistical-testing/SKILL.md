---
name: lc-ms-feature-statistical-testing
description: Use when the input is a preprocessed MetaboSet object (or imputed feature
  abundance matrix with group labels) and the goal is per-feature group comparisons
  — selecting which molecular features differ between two or more study groups with
  multiplicity-corrected p-values.
when_to_use_negative:
- The data have not yet been preprocessed (missing values not marked, drift not corrected,
  no quality flags or imputation) — run the notame preprocessing steps first
- The question is multivariate (e.g. PCA, PLS or clustering) rather than feature-wise
  univariate group comparison
- The design has no explicit grouping variable, so group-wise tests like Welch's t-test
  or ANOVA are not applicable
edam_operation: http://edamontology.org/operation_3650
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: R package providing perform_oneway_anova and perform_pairwise_t_test for feature-wise
    Welch's ANOVA and post-hoc pairwise Welch's t-tests on MetaboSet objects
  repo: https://github.com/hanhineva-lab/notame
- name: R
  role: Environment in which notame's statistical functions are run; dplyr::left_join
    and join_fData combine and attach results
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:4219ab83fb7b22c912a3c9f1d7d1404470c82b025372ede30eb7823dab0cbaae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_3650
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# lc-ms-feature-statistical-testing

## Summary

Feature-wise univariate testing of LC–MS metabolomics feature tables within notame: Welch's t-test (or Mann–Whitney U) for two-group designs, Welch's one-way ANOVA with post-hoc pairwise Welch's t-tests for multi-group designs, and Benjamini–Hochberg FDR correction yielding per-feature q-values.

## When to use

Use when the input is a preprocessed MetaboSet object (or imputed feature abundance matrix with group labels) and the goal is per-feature group comparisons — selecting which molecular features differ between two or more study groups with multiplicity-corrected p-values.

## When NOT to use

- The data have not yet been preprocessed (missing values not marked, drift not corrected, no quality flags or imputation) — run the notame preprocessing steps first
- The question is multivariate (e.g. PCA, PLS or clustering) rather than feature-wise univariate group comparison
- The design has no explicit grouping variable, so group-wise tests like Welch's t-test or ANOVA are not applicable

## Inputs

- Preprocessed MetaboSet object holding the feature abundance matrix, sample metadata and feature metadata (from MS-DIAL peak table)
- Grouping variable in sample data defining the two or more study groups being compared
- Formula string such as 'Feature ~ Group' specifying the model for ANOVA

## Outputs

- Data frame of per-feature ANOVA_P and ANOVA_P_FDR (Benjamini–Hochberg adjusted) values
- Post-hoc pairwise Welch's t-test results for features passing the FDR cutoff
- Combined statistical table attached to MetaboSet fData via join_fData, written to disk

## How to apply

Load the imputed, quality-flagged MetaboSet (e.g. produced from an MS-DIAL peak table via read_from_excel, mark_nas, correct_drift, flag_quality and missForest random-forest imputation). For a two-group design, apply Welch's t-test feature-wise (Mann–Whitney U as the non-parametric alternative); for three or more groups, run notame's perform_oneway_anova with formula_char = 'Feature ~ Group' (Welch's ANOVA) on each feature, then correct all ANOVA p-values with Benjamini–Hochberg FDR, recording ANOVA_P and ANOVA_P_FDR per feature. Select features passing the FDR threshold (e.g. ANOVA_P_FDR < 0.2) and run post-hoc pairwise Welch's t-tests with perform_pairwise_t_test(group = 'Group'). Join the ANOVA and pairwise results (dplyr::left_join) and attach them to the object's feature data with join_fData before writing the combined per-feature table.

## Related tools

- **notame** (R package providing perform_oneway_anova and perform_pairwise_t_test for feature-wise Welch's ANOVA and post-hoc pairwise Welch's t-tests on MetaboSet objects) — https://github.com/hanhineva-lab/notame
- **R** (Environment in which notame's statistical functions are run; dplyr::left_join and join_fData combine and attach results)

## Examples

```
anova_res <- perform_oneway_anova(mset, formula_char = 'Feature ~ Group'); pw_res <- perform_pairwise_t_test(mset, group = 'Group')
```

## Evaluation signals

- Every feature in the input MetaboSet has a non-NA ANOVA_P and a corresponding Benjamini–Hochberg ANOVA_P_FDR value
- The q-value distribution is monotone: ANOVA_P_FDR >= ANOVA_P for all features
- Only features passing the FDR threshold (e.g. ANOVA_P_FDR < 0.2) appear in the post-hoc pairwise test table
- The joined per-feature result table has one row per feature with no duplicated feature IDs after left_join of ANOVA and pairwise results
- The combined table is attached to fData via join_fData and the written output contains the same number of rows as there are features

## Limitations

- notame's package API is still experimental and breaking changes are possible between versions
- The Benjamini–Hochberg adjustment assumes appropriate dependence structure among feature p-values; the article specifies FDR but does not prescribe a single universal significance cutoff, so the threshold (e.g. 0.2) is analysis-specific
- Statistical analysis is only meaningful after the full preprocessing chain (missing-value handling, drift correction, quality flagging, imputation); tests on raw peak tables are invalid

## Evidence

- [results] For two-group designs Welch's t-test is used (with Mann-Whitney U as non-parametric alternative), multi-group designs use Welch's one-way ANOVA with post-hoc pairwise Welch's t-tests: [evidence span withheld — anchor status: not_found]
- [results] p-values are adjusted for multiple testing using the Benjamini–Hochberg FDR approach, yielding q-values per feature: [evidence span withheld — anchor status: not_found]
- [intro] notame can be used to analyze preprocessed LC-MS data in non-targeted metabolomics: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
- [readme] statistical analyses, both feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [readme] Preprocessing prerequisites: missing values, drift, quality flags and imputation precede testing: [evidence span withheld — anchor status: external_doc]
