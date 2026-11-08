---
name: benjamini-hochberg-multiple-testing-correction
description: Use when a metabolomics (or other omics) feature table requires a per-feature
  significance test (e.g. linear regression for diet/group effect) across thousands
  of features and raw p-values must be adjusted before selecting differential features
  with cutoffs such as Log2FC > 1.5 and adj-p < 0.05.
when_to_use_negative:
- When a small pre-specified number of pre-registered hypotheses is being tested (no
  multiplicity from thousands of features) — apply raw p-value thresholds instead
- When the goal is exploratory visualization of unadjusted p-values without any candidate
  selection (BH correction will shrink the displayed signal)
- When the dataset is not a feature-level table requiring per-feature tests (e.g.
  single-sample clinical correlation already summarized elsewhere)
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: dbnorm
  role: Provides the batch-effect-corrected metabolomics feature tables (ber, ComBat,
    lowess) on which the per-feature model, BH adjustment, and volcano plot are performed
  repo: https://github.com/NBDZ/dbnorm
- name: R
  role: Analysis environment used to fit per-feature linear regressions, perform Benjamini–Hochberg
    adjustment, and generate volcano plots
- name: ggplot2
  role: Used in R to draw the volcano plot of Log2FC against -log10(adjusted p-value)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-84824-3
    title: DBnorm as an R package for the comparison and selection of appropriate
      statistical methods for batch effect correction in metabolomic studies
  source_content_hash: sha256:879dde67a3e32a1df6d895ee74042f9fc74dabba44e174a08bc18faabe6ee0f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NBDZ/dbnorm
  tools:
  - R
  - dbnorm
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Control the false discovery rate across the thousands of metabolic features tested in a batch-effect-corrected metabolomics dataset by adjusting raw p-values with the Benjamini–Hochberg procedure, as done for the 8-week HFD vs control sc-AT comparison in dbnorm's differential analysis.

## When to use

Use when a metabolomics (or other omics) feature table requires a per-feature significance test (e.g. linear regression for diet/group effect) across thousands of features and raw p-values must be adjusted before selecting differential features with cutoffs such as Log2FC > 1.5 and adj-p < 0.05.

## When NOT to use

- When a small pre-specified number of pre-registered hypotheses is being tested (no multiplicity from thousands of features) — apply raw p-value thresholds instead
- When the goal is exploratory visualization of unadjusted p-values without any candidate selection (BH correction will shrink the displayed signal)
- When the dataset is not a feature-level table requiring per-feature tests (e.g. single-sample clinical correlation already summarized elsewhere)

## Inputs

- Feature-level raw p-values and log2 fold changes from a per-feature model (e.g. linear regression of each metabolic feature on diet group)
- Batch-effect-corrected (e.g. ber) or raw UPLC-HRMS metabolomics feature table with samples × features
- Group labels defining the comparison of interest (e.g. 8-week HFD vs chow control)

## Outputs

- Adjusted (BH) p-value per feature
- List of significant metabolites meeting Log2FC > 1.5 and adj-p < 0.05 (exported to CSV)
- Volcano plot (Log2FC vs -log10 adjusted p-value) highlighting the significant subset (exported to PNG)

## How to apply

Fit a per-feature statistical model — in the article's workflow, a linear regression model estimating the fold change and p-value of the diet effect for each metabolic feature in the UPLC-HRMS sc-AT comparison. Collect the raw p-values from all features and apply the Benjamini–Hochberg FDR adjustment across the full family of tests (features × conditions). Then define differential metabolites using the joint criteria Log2FC > 1.5 AND adjusted p-value < 0.05, and use the adjusted (not raw) p-value as the y-axis (-log10 scale) of the volcano plot so the highlighted subset reflects FDR control. Rationale: with >20 thousand m/z values tested simultaneously in the untargeted LC-MS data, raw p-values yield an inflated candidate list, so BH correction defines which metabolites are significantly affected by HFD only after batch correction.

## Related tools

- **dbnorm** (Provides the batch-effect-corrected metabolomics feature tables (ber, ComBat, lowess) on which the per-feature model, BH adjustment, and volcano plot are performed) — https://github.com/NBDZ/dbnorm
- **R** (Analysis environment used to fit per-feature linear regressions, perform Benjamini–Hochberg adjustment, and generate volcano plots)
- **ggplot2** (Used in R to draw the volcano plot of Log2FC against -log10(adjusted p-value))

## Examples

```
f <- ber_corrected_table; pvals <- apply(f, 2, function(x) summary(lm(x ~ group))$coef[2,4]); padj <- p.adjust(pvals, method="BH"); sig <- which(log2FC > 1.5 & padj < 0.05)
```

## Evaluation signals

- Adjusted p-values must be monotonically non-decreasing relative to sorted raw p-values (BH step-up property) and all adj-p values ≤ 1
- Number of significant features (Log2FC > 1.5 AND adj-p < 0.05) should be ≤ the number passing the same Log2FC cutoff with raw p < 0.05, and should shrink or change after correction
- In the article's case, the ber-corrected data should recover HFD-affected metabolites (mostly lipids) that are not significant in raw data, confirming FDR correction on the corrected table
- Volcano plot y-axis must plot -log10(adjusted p-value), not -log10(raw p-value)
- CSV export must contain exactly the features satisfying both cutoffs simultaneously

## Limitations

- BH adjustment is performed on the full family of features; in dbnorm's exploratory pipeline (Visodbnorm/dbnormSCORE) it is recommended for < 2000 features for computational speed
- The significance threshold combination (Log2FC > 1.5, adj-p < 0.05) is dataset-specific and not a universal default
- The article does not specify a dbnorm software version or exact implementation details of the BH call; the exact R function used is not documented

## Evidence

- [other] Adjust p-values using Benjamini-Hochberg (BH) correction for multiple testing: [evidence span withheld — anchor status: not_found]
- [results] Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level.: "Significant criteria of Log2FC > 1.5 and adj-p-value < 0.05 are considered to draw a list of metabolites with differential level."
- [results] Data processing using XCMS software, yielded more than 20 thousand m/z values: "Data processing using XCMS software (https ://xcmso nline .scrip ps.edu/), yielded more than 20 thousand m/z values"
- [readme] This function is suggested for less than 2000 features (variables).: "This function is suggested for less than 2000 features (variables)."
- [other] ber-corrected data yielded a more exhaustive candidate list for HFD effects than raw data: [evidence span withheld — anchor status: not_found]
