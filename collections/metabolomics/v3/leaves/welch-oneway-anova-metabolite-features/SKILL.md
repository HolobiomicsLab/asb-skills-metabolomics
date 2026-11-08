---
name: welch-oneway-anova-metabolite-features
description: Use when a preprocessed MetaboSet peak table from MS-DIAL via notame
  must be tested feature-wise across two or more study groups, and per-feature p-values
  need multiple-testing correction.
when_to_use_negative:
- The data has not yet been preprocessed — missing values not marked, drift not corrected
  with correct_drift, low-quality features not flagged with flag_quality, or missing
  values not imputed (random-forest imputation recommended)
- The research question calls for multivariate modelling or quality-control visualization
  rather than feature-wise hypothesis testing
- The input is still raw LC–MS instrument files — peak picking (e.g. with MS-DIAL)
  must precede any notame statistical testing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: notame
  role: R package providing perform_oneway_anova, perform_pairwise_t_test and join_fData
    for feature-wise statistical testing on MetaboSet objects
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Runtime environment in which notame's statistical testing functions are executed
    (R >= 3.5.0)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:aee0f3ee1187c4d231c6deb67aa804ab17e51de637b84e37d531be7a900e2d27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# welch-oneway-anova-metabolite-features

## Summary

Feature-wise univariate testing for non-targeted LC–MS metabolomics in the notame workflow: Welch's one-way ANOVA per molecular feature across study groups (Welch's t-test for two-group designs), followed by Benjamini–Hochberg FDR correction producing per-feature q-values.

## When to use

Use when a preprocessed MetaboSet peak table from MS-DIAL via notame must be tested feature-wise across two or more study groups, and per-feature p-values need multiple-testing correction.

## When NOT to use

- The data has not yet been preprocessed — missing values not marked, drift not corrected with correct_drift, low-quality features not flagged with flag_quality, or missing values not imputed (random-forest imputation recommended)
- The research question calls for multivariate modelling or quality-control visualization rather than feature-wise hypothesis testing
- The input is still raw LC–MS instrument files — peak picking (e.g. with MS-DIAL) must precede any notame statistical testing

## Inputs

- Preprocessed MetaboSet object holding the feature abundance matrix and feature/sample annotation (originating from an MS-DIAL Excel peak table, e.g. sample_data_whole.xlsx)
- Sample grouping variable (e.g. 'Group') defining two or more study groups in pData

## Outputs

- Per-feature results data frame with ANOVA_P and ANOVA_P_FDR (Benjamini–Hochberg q-values) columns
- Post-hoc pairwise Welch's t-test results for features with ANOVA_P_FDR < 0.2
- Combined statistical table joined onto the MetaboSet fData and written to disk

## How to apply

Load the imputed MetaboSet object (built from the MS-DIAL Excel peak table via read_from_excel, after mark_nas, correct_drift, flag_quality and missForest random-forest imputation). For multi-group designs, fit Welch's one-way ANOVA per feature with notame's perform_oneway_anova using formula_char = 'Feature ~ Group'; for two-group designs use Welch's t-test (Mann–Whitney U as the non-parametric alternative). Apply Benjamini–Hochberg FDR correction to record ANOVA_P and ANOVA_P_FDR per feature, select features with ANOVA_P_FDR < 0.2, and run post-hoc pairwise Welch's t-tests on them with perform_pairwise_t_test(group = 'Group'). Combine ANOVA and pairwise results with dplyr::left_join and attach them to the MetaboSet feature data via join_fData before writing the per-feature statistical table to disk.

## Related tools

- **notame** (R package providing perform_oneway_anova, perform_pairwise_t_test and join_fData for feature-wise statistical testing on MetaboSet objects) — https://github.com/antonvsdata/notame
- **R** (Runtime environment in which notame's statistical testing functions are executed (R >= 3.5.0))

## Examples

```
anova_res <- perform_oneway_anova(mset, formula_char = 'Feature ~ Group'); anova_res <- perform_pairwise_t_test(anova_res, group = 'Group'); anova_res <- join_fData(anova_res, mset)
```

## Evaluation signals

- The results data frame contains both ANOVA_P and ANOVA_P_FDR columns, one row per molecular feature
- Every ANOVA_P_FDR value is >= the corresponding ANOVA_P (BH adjustment never decreases p-values), and FDR-adjusted values fall in [0, 1]
- Pairwise Welch's t-test results exist only for features passing the ANOVA_P_FDR < 0.2 selection threshold
- The combined table (ANOVA + left_join'ed pairwise tests) is successfully attached to fData via join_fData, i.e. row names match the MetaboSet feature IDs
- All features in the input MetaboSet exprs matrix appear in the output statistical table (no features silently dropped by the test)

## Limitations

- The notame package API is still quite experimental and breaking changes are possible; the original antonvsdata/notame repository is migrated, so issues/PRs belong at hanhineva-lab/notame
- Welch's t-test/ANOVA are parametric tests; Mann–Whitney U is the non-parametric alternative for two-group designs
- The feature-wise tests assume the input has already passed notame quality control — datasets contain a significant number of low-quality features that should be flagged before testing

## Evidence

- [readme] A bunch of statistical analyses, both feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [intro] This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics.: "This package can be used to analyze preprocessed LC-MS data in non-targeted metabolomics"
- [methods] flag_quality flags features based on quality metrics before testing: "```flag_quality``` is used to flag features based on the other quality metrics"
