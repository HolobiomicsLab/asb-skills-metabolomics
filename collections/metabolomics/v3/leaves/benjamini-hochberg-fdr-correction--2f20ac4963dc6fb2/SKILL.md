---
name: benjamini-hochberg-fdr-correction
description: Use when you have feature-wise p-values from univariate tests (Welch's
  t-test, Mann-Whitney U, or one-way ANOVA) across a large LC-MS metabolomics feature
  table and need multiplicity-corrected q-values to select candidate biomarkers.
when_to_use_negative:
- Input is already a list of corrected q-values or an FDR-adjusted table — re-correcting
  would be redundant and would not double up but would misalign columns.
- The analysis goal is unsupervised multivariate modelling (PCA, clustering, correlation-based
  feature grouping) rather than hypothesis testing — no p-values are generated so
  FDR correction does not apply.
- Only one or a few features are being tested in isolation (single targeted compound
  confirmation) — the multiplicity problem Benjamini-Hochberg addresses does not arise.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: notame
  role: R package supplying the MetaboSet object model and the feature-wise test functions
    (perform_oneway_anova, perform_pairwise_t_test) whose p-value outputs are FDR-corrected,
    plus join_fData for attaching adjusted results back to feature metadata
  repo: https://github.com/antonvsdata/notame
- name: R
  role: Runtime environment in which notame's FDR correction and downstream statistical
    functions are executed; notame requires R >= 3.5.0
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo10040135
    title: '“Notame”: Workflow for Non-Targeted LC–MS Metabolic Profiling'
  source_content_hash: sha256:9a3ad351c8d57dc610ff945b4d5a5cdcb9b56633503faf31372cf585344765e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/antonvsdata/notame
  tools:
  - R
  - notame
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# benjamini-hochberg-fdr-correction

## Summary

Apply Benjamini-Hochberg false discovery rate (FDR) correction to feature-wise p-values from Welch's t-tests or one-way ANOVA in the notame LC-MS metabolomics pipeline, producing per-feature q-values that control the proportion of false positives across hundreds of molecular features.

## When to use

Use when you have feature-wise p-values from univariate tests (Welch's t-test, Mann-Whitney U, or one-way ANOVA) across a large LC-MS metabolomics feature table and need multiplicity-corrected q-values to select candidate biomarkers.

## When NOT to use

- Input is already a list of corrected q-values or an FDR-adjusted table — re-correcting would be redundant and would not double up but would misalign columns.
- The analysis goal is unsupervised multivariate modelling (PCA, clustering, correlation-based feature grouping) rather than hypothesis testing — no p-values are generated so FDR correction does not apply.
- Only one or a few features are being tested in isolation (single targeted compound confirmation) — the multiplicity problem Benjamini-Hochberg addresses does not arise.

## Inputs

- Imputed, preprocessed MetaboSet object (e.g. loaded from sample_data_whole.xlsx via notame's read_from_excel and processed with mark_nas, correct_drift, flag_quality, and missForest impute_rf)
- Vector of raw feature-wise p-values (ANOVA_P or t-test p-values) from notame's perform_oneway_anova, perform_pairwise_t_test, or equivalent univariate test functions
- Group/phenotype label column (e.g. 'Group') in the sample metadata for defining test contrasts

## Outputs

- Per-feature results data frame with added ANOVA_P_FDR (Benjamini-Hochberg adjusted q-value) column alongside the raw ANOVA_P column
- Subset of features passing the q-value threshold (e.g. ANOVA_P_FDR < 0.2) selected for follow-up post-hoc pairwise testing
- Combined feature-wise statistical table (ANOVA + pairwise Welch's t-test results) attached to MetaboSet fData and written to disk

## How to apply

In the notame workflow, feature-wise univariate testing generates one p-value per molecular feature (Welch's t-test or Mann-Whitney U for two-group designs; Welch's one-way ANOVA for multi-group designs), but running hundreds of tests in parallel inflates the false discovery rate. Correct this by applying the Benjamini-Hochberg FDR procedure to the vector of p-values, storing the result as ANOVA_P_FDR alongside the raw ANOVA_P column in the per-feature results data frame. The corrected q-values are then used as the selection threshold — in the notame example workflow, features with ANOVA_P_FDR < 0.2 are selected for post-hoc pairwise Welch's t-tests with perform_pairwise_t_test(group = 'Group'). The combined ANOVA and pairwise statistics table is attached to the MetaboSet feature data (fData) via join_fData and written to disk as the canonical output. This FDR correction step sits downstream of preprocessing (mark_nas, correct_drift, flag_quality, missForest random-forest imputation) and is applied before any multivariate or visualisation analysis.

## Related tools

- **notame** (R package supplying the MetaboSet object model and the feature-wise test functions (perform_oneway_anova, perform_pairwise_t_test) whose p-value outputs are FDR-corrected, plus join_fData for attaching adjusted results back to feature metadata) — https://github.com/antonvsdata/notame
- **R** (Runtime environment in which notame's FDR correction and downstream statistical functions are executed; notame requires R >= 3.5.0)

## Examples

```
results <- perform_oneway_anova(metaboset, formula_char = 'Feature ~ Group'); results$ANOVA_P_FDR <- p.adjust(results$ANOVA_P, method = 'BH')
```

## Evaluation signals

- The results data frame contains both ANOVA_P and ANOVA_P_FDR columns with equal length matching the number of tested features.
- All values in ANOVA_P_FDR are ≥ the corresponding raw ANOVA_P (FDR adjustment never decreases p-values).
- ANOVA_P_FDR values are bounded in [0, 1] and sorted/scaled consistently with the BH step-up procedure applied over the full p-value vector.
- Downstream feature selection uses the FDR column — the post-hoc pairwise test set is exactly the subset with ANOVA_P_FDR below the chosen cutoff (0.2 in the notame example).
- The combined statistical table is verifiably attached to fData (row count unchanged; new ANOVA/P_FDR columns present) before export.

## Limitations

- The Benjamini-Hochberg procedure in notame's example is applied at a generous threshold (ANOVA_P_FDR < 0.2) — features near the cutoff require post-hoc pairwise confirmation and biological interpretation, not standalone acceptance.
- The notame package API is explicitly still experimental and breaking changes are possible, so the exact function names for FDR correction may shift between versions.
- BH-FDR assumes p-values are independent or positively correlated; strong feature-feature correlations from co-eluting compounds (common in LC-MS where one compound yields multiple molecular features) may make the correction conservative or non-optimal.

## Evidence

- [methods] BH-FDR is applied to feature-wise p-values with explicit threshold: [evidence span withheld — anchor status: not_found]
- [full_text] FDR correction is integrated into the notame feature-wise testing workflow: [evidence span withheld — anchor status: not_found]
- [readme] notame supports feature-wise tests and multivariate models: [evidence span withheld — anchor status: external_doc]
- [readme] Downstream results are attached back to the feature data structure: [evidence span withheld — anchor status: external_doc]
