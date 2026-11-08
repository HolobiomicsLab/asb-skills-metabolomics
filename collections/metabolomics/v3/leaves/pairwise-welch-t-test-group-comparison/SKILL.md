---
name: pairwise-welch-t-test-group-comparison
description: Use when an imputed MetaboSet from LC–MS metabolomics has two or more
  sample groups in the Group annotation and you need per-feature univariate statistics
  — e.g. ANOVA_P_FDR-selected features for post-hoc pairwise testing, or direct two-group
  comparisons.
when_to_use_negative:
- Input is still a raw instrument file or an unimputed MS-DIAL peak table — preprocessing
  (mark_nas, correct_drift, flag_quality, imputation) must come first.
- The analysis goal is multivariate (e.g. PCA/OPLS) rather than feature-wise univariate
  testing.
- Features are flagged low-quality by flag_quality/flag_detection — testing flagged
  features is not the intended use.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: notame
  role: Provides perform_pairwise_t_test and perform_oneway_anova for the feature-wise
    Welch's t-test and ANOVA/FDR pipeline on MetaboSet objects
  repo: https://github.com/hanhineva-lab/notame
- name: R
  role: Runtime environment in which notame functions, dplyr::left_join and write-table
    utilities execute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:8fd888f71ae9f8efcf7e15636e2671ef4d06822601f630b0dbde16d978f752f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hanhineva-lab/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pairwise-welch-t-test-group-comparison

## Summary

Feature-wise pairwise group comparison in the notame LC–MS metabolomics workflow: for each molecular feature, run Welch's two-sample t-test (Mann–Whitney U as the non-parametric alternative) between groups, producing feature-level p-values that are subsequently Benjamini–Hochberg FDR-corrected.

## When to use

Use when an imputed MetaboSet from LC–MS metabolomics has two or more sample groups in the Group annotation and you need per-feature univariate statistics — e.g. ANOVA_P_FDR-selected features for post-hoc pairwise testing, or direct two-group comparisons.

## When NOT to use

- Input is still a raw instrument file or an unimputed MS-DIAL peak table — preprocessing (mark_nas, correct_drift, flag_quality, imputation) must come first.
- The analysis goal is multivariate (e.g. PCA/OPLS) rather than feature-wise univariate testing.
- Features are flagged low-quality by flag_quality/flag_detection — testing flagged features is not the intended use.

## Inputs

- Preprocessed MetaboSet object holding the LC–MS feature abundance matrix and sample metadata with a Group column (built from an MS-DIAL Excel peak table)
- Design formula string for multi-group context, e.g. 'Feature ~ Group'

## Outputs

- Per-feature pairwise Welch's t-test p-values (and optionally Mann–Whitney U p-values) joined with ANOVA results
- A feature-wise statistics data frame attached to MetaboSet fData via join_fData, writable to disk

## How to apply

Start from a preprocessed, imputed MetaboSet object built from the MS-DIAL peak table (e.g. sample_data_whole.xlsx read via read_from_excel, then mark_nas, correct_drift, flag_quality, and random-forest imputation). For two-group designs call notame's feature-wise test with Group as the grouping variable to obtain Welch's t-test p-values per molecular feature; the non-parametric alternative is Mann–Whitney U for features violating normality assumptions. For multi-group designs, first run perform_oneway_anova with formula_char = 'Feature ~ Group' and select features meeting the ANOVA_P_FDR < 0.2 threshold, then run perform_pairwise_t_test(group = 'Group') on those selected features to get the pairwise Welch's t-test contrasts. Combine the pairwise results with the ANOVA results using dplyr::left_join and attach them to the MetaboSet feature data with join_fData so the p-values travel with each feature's annotations, then write the combined table to disk as the canonical output file.

## Related tools

- **notame** (Provides perform_pairwise_t_test and perform_oneway_anova for the feature-wise Welch's t-test and ANOVA/FDR pipeline on MetaboSet objects) — https://github.com/hanhineva-lab/notame
- **R** (Runtime environment in which notame functions, dplyr::left_join and write-table utilities execute)

## Examples

```
pairwise_results <- perform_pairwise_t_test(metaboset, group = 'Group'); metaboset <- join_fData(metaboset, pairwise_results)
```

## Evaluation signals

- ANOVA_P and ANOVA_P_FDR columns are present for every tested molecular feature after the run
- Only features with ANOVA_P_FDR < 0.2 receive pairwise Welch's t-test rows
- Adjusted q-values differ from raw p-values, confirming Benjamini–Hochberg FDR was applied
- The combined results table row count matches the number of pairwise Group contrasts × selected features
- The final feature statistics are attached to MetaboSet fData (verifiable via join_fData output) before writing to disk

## Limitations

- The package API is still quite experimental and breaking changes are possible.
- Welch's t-test assumes approximate normality; Mann–Whitney U is the non-parametric alternative the article names, so pick it when distribution assumptions fail.
- The ANOVA_P_FDR < 0.2 cutoff used for feature selection before pairwise testing is looser than a conventional 0.05 threshold — confirm this suits your dataset before reuse.

## Evidence

- [results] Welch's t-test as default two-group test; BH FDR as correction: [evidence span withheld — anchor status: not_found]
- [results] Feature selection threshold for pairwise testing: [evidence span withheld — anchor status: not_found]
- [results] ANOVA call with formula and groups: [evidence span withheld — anchor status: not_found]
- [results] Results joining and attachment to MetaboSet: [evidence span withheld — anchor status: not_found]
- [readme] notame provides feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [abstract] Article scope of the workflow: "We provide an overview of lab protocols and statistical methods that we commonly practice for the analysis of nutritional metabolomics data."
