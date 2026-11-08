---
name: multiassayexperiment-statistics-extraction
description: Use when a MetaboDiff analysis has produced a MultiAssayExperiment object
  and you need the per-metabolite p-values, Benjamini-Hochberg adjusted p-values,
  and difference-in-means (dm) for downstream tables or volcano plots.
when_to_use_negative:
- When more than two groups are being compared without wanting ANOVA — MetaboDiff
  switches from Student's T-Test to ANOVA, so dm interpretation differs
- When the input is a raw metabolite measurement table rather than a preprocessed
  MultiAssayExperiment object — the create_mae/normalization/imputation steps must
  run first
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetaboDiff
  role: Provides diff_test, which computes per-metabolite statistics (Student's T-Test
    or ANOVA) and stores them in the MultiAssayExperiment metadata slot
  repo: http://github.com/andreasmock/MetaboDiff/
- name: MultiAssayExperiment
  role: Container object created by create_mae that holds the metabolite assays (raw,
    imputed, norm, norm_imputed) and the metadata slot with the statistics dataframes
- name: R
  role: Runtime environment; MetaboDiff requires R version 4.0.2 or higher
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty344
    title: 'MetaboDiff: an R package for differential metabolomic analysis'
  source_content_hash: sha256:635e80cc5dc5aa093029dc5ec92c89eea2e7c33e21527ef2ff3e1a5fb3715d49
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# multiassayexperiment-statistics-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-metabolite differential statistics (pval, adj_pval, dm) from the metadata slot of a MetaboDiff MultiAssayExperiment object after running diff_test, so downstream plots and tables can be built from the t-test/ANOVA results.

## When to use

Use when a MetaboDiff analysis has produced a MultiAssayExperiment object and you need the per-metabolite p-values, Benjamini-Hochberg adjusted p-values, and difference-in-means (dm) for downstream tables or volcano plots.

## When NOT to use

- When more than two groups are being compared without wanting ANOVA — MetaboDiff switches from Student's T-Test to ANOVA, so dm interpretation differs
- When the input is a raw metabolite measurement table rather than a preprocessed MultiAssayExperiment object — the create_mae/normalization/imputation steps must run first

## Inputs

- Normalized, imputed MultiAssayExperiment object from the MetaboDiff pipeline (data slots: raw, imputed, norm, norm_imputed)
- Group factor column(s) in the sample metadata, e.g. tumor_normal or random_gender

## Outputs

- Statistics dataframe(s) in the MultiAssayExperiment metadata slot with columns pval, adj_pval, and dm
- Per-metabolite Benjamini-Hochberg adjusted p-values and difference-in-means between group levels

## How to apply

Load the normalized, imputed MultiAssayExperiment object produced by the MetaboDiff preprocessing pipeline (data slots: raw, imputed, norm, norm_imputed) and run diff_test with group_factors such as c("tumor_normal","random_gender"); MetaboDiff applies the Student's T-Test for two groups and ANOVA for more than two groups, correcting p-values with the Benjamini-Hochberg procedure. Extract the resulting statistics dataframes from the metadata slot and verify they contain the columns pval, adj_pval, and dm. Note that the first factor level is the reference for the dm calculation, so the sign of dm depends on the chosen reference group. These extracted dataframes feed volcano_plot rendering, where significant metabolites are those with adjusted p-value < 0.05 and |dm| > 0.5.

## Related tools

- **MetaboDiff** (Provides diff_test, which computes per-metabolite statistics (Student's T-Test or ANOVA) and stores them in the MultiAssayExperiment metadata slot) — http://github.com/andreasmock/MetaboDiff/
- **MultiAssayExperiment** (Container object created by create_mae that holds the metabolite assays (raw, imputed, norm, norm_imputed) and the metadata slot with the statistics dataframes)
- **R** (Runtime environment; MetaboDiff requires R version 4.0.2 or higher)

## Examples

```
metabolite_data <- diff_test(metabolite_data, group_factors = c("tumor_normal","random_gender")); stats <- metadata(metabolite_data)[["diff_test"]]
```

## Evaluation signals

- The extracted statistics dataframes contain the columns pval, adj_pval, and dm
- adj_pval values match the Benjamini-Hochberg correction of the raw pval column
- dm values reflect the difference in means with the first factor level as reference (sign flips if the reference is changed)
- Sanity check: a random grouping comparison (e.g. random_gender) yields no metabolites significant after Benjamini-Hochberg correction
- Volcano plots colored against dashed thresholds (adjusted p-value < 0.05, |dm| > 0.5) highlight the same metabolites ranked significant in the extracted table

## Limitations

- dm sign depends on the first factor level being the reference; misordering group_factors changes the interpretation
- Statistical model is fixed by group count: Student's T-Test for two groups, ANOVA for more than two — no user-selected alternative test
- Statistics are only meaningful after the full preprocessing pipeline (kNN imputation with 0.4 cutoff, vsn normalization, cluster removal) has been applied

## Evidence
