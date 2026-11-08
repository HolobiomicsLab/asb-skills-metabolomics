---
name: volcano-plot-interpretation-metabolomics
description: Use when a two-group metabolomics comparison (e.g. tumor vs normal) has
  been tested per metabolite and you need to display and interpret effect size (dm)
  versus significance (pval or adj_pval) in a volcano plot.
when_to_use_negative:
- More than two groups are compared — the diff_test function applies ANOVA rather
  than the Student's T-Test, so a volcano plot of two-group dm is not appropriate
- The aim is exploring sample traits in the data-derived metabolic correlation network
  rather than two-group differential testing
- The input is a raw metabolite measurement table that has not yet been imputed (kNN,
  cutoff 0.4) and variance-stabilizing normalized (vsn)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MetaboDiff
  role: Provides diff_test (Student's T-Test + Benjamini-Hochberg correction) and
    volcano_plot for rendering the two-group comparison
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Data container (created by create_mae) holding the metabolite measurements
    and group factors consumed by diff_test and volcano_plot
- name: R
  role: Runtime environment for MetaboDiff (requires R version 4.0.2 or higher)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:50c379d6d9b4fbfdcbcdfd3eab73c29c1ff1449620adc4edd9cb4bc16904f7fa
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Volcano plot interpretation for differential metabolomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualizes the per-metabolite statistical results of a two-group differential metabolomics comparison as a volcano plot, plotting difference in means (dm) against raw or Benjamini-Hochberg adjusted p-values, with significant metabolites highlighted relative to fixed significance thresholds.

## When to use

Use when a two-group metabolomics comparison (e.g. tumor vs normal) has been tested per metabolite and you need to display and interpret effect size (dm) versus significance (pval or adj_pval) in a volcano plot.

## When NOT to use

- More than two groups are compared — the diff_test function applies ANOVA rather than the Student's T-Test, so a volcano plot of two-group dm is not appropriate
- The aim is exploring sample traits in the data-derived metabolic correlation network rather than two-group differential testing
- The input is a raw metabolite measurement table that has not yet been imputed (kNN, cutoff 0.4) and variance-stabilizing normalized (vsn)

## Inputs

- Normalized, imputed metabolite MultiAssayExperiment object (data slots: raw, imputed, norm, norm_imputed)
- Group factor annotations (e.g. tumor_normal, random_gender) from the MAE colData
- Statistics dataframes from diff_test with columns pval, adj_pval, dm

## Outputs

- Volcano plot of dm versus p-value (unadjusted, p_adjust = FALSE)
- Volcano plot of dm versus Benjamini-Hochberg adjusted p-value (p_adjust = TRUE)
- List of significant metabolites passing adj_pval < 0.05 and |dm| > dm_cutoff (0.5)

## How to apply

Run MetaboDiff's diff_test on the normalized, imputed MultiAssayExperiment object (data slots: raw, imputed, norm, norm_imputed) with the desired group_factors, e.g. c("tumor_normal","random_gender"); the function applies the Student's T-Test for two groups and corrects p-values by the Benjamini-Hochberg procedure. Extract the resulting statistics dataframes from the metadata slot and verify they contain the columns pval, adj_pval, and dm, remembering that the first factor level is the reference for the dm calculation. Render the comparison with volcano_plot, using label_colors such as c("darkseagreen","dodgerblue") and a dm_cutoff of 0.5; produce both an unadjusted (p_adjust = FALSE) and a BH-adjusted (p_adjust = TRUE) version. Significant metabolites are colored relative to the dashed significance thresholds (adjusted p-value < 0.05 and |dm| > 0.5), so interpret points in the upper corners as the biologically and statistically meaningful hits.

## Related tools

- **MetaboDiff** (Provides diff_test (Student's T-Test + Benjamini-Hochberg correction) and volcano_plot for rendering the two-group comparison) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Data container (created by create_mae) holding the metabolite measurements and group factors consumed by diff_test and volcano_plot)
- **R** (Runtime environment for MetaboDiff (requires R version 4.0.2 or higher))

## Examples

```
volcano_plot(mae, group_factor = "tumor_normal", label_colors = c("darkseagreen","dodgerblue"), dm_cutoff = 0.5, p_adjust = TRUE)
```

## Evaluation signals

- The diff_test statistics dataframe contains the columns pval, adj_pval, and dm
- Significant metabolites are colored relative to the dashed thresholds at adjusted p-value < 0.05 and |dm| > 0.5
- The random grouping comparison (random_gender) shows no metabolites significant after Benjamini-Hochberg correction (sanity check)
- The adjusted (p_adjust = TRUE) plot yields equal or fewer significant hits than the unadjusted (p_adjust = FALSE) plot
- The first factor level is confirmed as the reference for the sign of dm in the plot

## Limitations

- dm cutoff of 0.5 and adj_pval < 0.05 thresholds are example choices from the tutorial and may need tuning per dataset
- The Benjamini-Hochberg correction can eliminate all significant metabolites in weak or random groupings, as shown by the random_gender sanity check
- Volcano plots only summarize two-group comparisons; multi-group designs fall back to ANOVA and cannot be shown as a simple two-sided dm volcano

## Evidence
