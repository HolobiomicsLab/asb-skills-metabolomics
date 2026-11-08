---
name: multiple-testing-padjust-correction
description: Use when a metabolite abundance table (e.g. from Metabolomics Workbench
  study ST000915) has been tested per metabolite between two groups and raw p-values
  must be adjusted before calling significantly altered metabolites.
when_to_use_negative:
- The comparison involves more than two groups or repeated measures — use MetENP's
  anova_ana instead of the two-group Student's t-test path in significant_met.
- The metabolite table already contains an adjusted p-value column from an upstream
  statistical workflow, so re-adjusting would double-correct the same tests.
- Metabolites lack replicate intensity measurements per group (no within-group variance),
  making a t-test and hence p-value adjustment inappropriate.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3520
tools:
- name: MetENP
  role: R package whose significant_met function performs the per-metabolite t-test
    and applies the selected p_adjust method (holm, hochberg, hommel, BH)
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Runtime environment in which significant_met.R is sourced (or the MetENP package
    installed via devtools) and the adjustment is executed
  repo: https://github.com/metabolomicsworkbench/MetENP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:92e4aa830ca9e4943d8fdc0e08a4c856a181921ca3a46c6128173c230193162f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - MetENP
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# multiple-testing-padjust-correction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Corrects per-metabolite p-values from the MetENP significant_met Student's t-test step for multiple testing using one of the holm, hochberg, hommel, or BH (fdr) adjustment methods, so that significantly altered metabolites are selected at a controlled false-positive rate.

## When to use

Use when a metabolite abundance table (e.g. from Metabolomics Workbench study ST000915) has been tested per metabolite between two groups and raw p-values must be adjusted before calling significantly altered metabolites.

## When NOT to use

- The comparison involves more than two groups or repeated measures — use MetENP's anova_ana instead of the two-group Student's t-test path in significant_met.
- The metabolite table already contains an adjusted p-value column from an upstream statistical workflow, so re-adjusting would double-correct the same tests.
- Metabolites lack replicate intensity measurements per group (no within-group variance), making a t-test and hence p-value adjustment inappropriate.

## Inputs

- Metabolite abundance table from Metabolomics Workbench (retrieved via getmwstudies(studyid, typedata))
- Metadata table with factor_col (grouping factor), sample_col, and the two comparison groups factor1/factor2
- Chosen p-value adjustment method: holm, hochberg, hommel, or BH
- Chosen NA-handling method: half_of_min, remove_NAs, or 50percent

## Outputs

- significant_metabolites.csv with per-metabolite identifier, fold change, log2 fold change, raw p-value, and adjusted p-value
- Retained set of significantly altered metabolites used downstream for metabolite-class and KEGG pathway enrichment scoring

## How to apply

After MetENP's significant_met computes per-metabolite fold change, log2 fold change, and raw Student's t-test p-values between factor1 and factor2 of factor_col, pass a p_adjust value of holm, hochberg, hommel, or BH (fdr) so the function returns an adjusted p-value column alongside the raw p-value. Before testing, handle missing intensities with the chosen NA-filtering variant: half_of_min (replace NAs with half the minimum value), remove_NAs (drop metabolites with NAs), or 50percent (drop metabolites with >50% NAs), since the adjustment assumes one test per retained metabolite. Retain the metabolites whose adjusted p-value meets the significance threshold and write the per-metabolite table (metabolite identifier, fold change, log2 fold change, p-value, adjusted p-value) to significant_metabolites.csv. For comparisons spanning more than two groups, MetENP provides the separate anova_ana function instead of the two-group t-test path.

## Related tools

- **MetENP** (R package whose significant_met function performs the per-metabolite t-test and applies the selected p_adjust method (holm, hochberg, hommel, BH)) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Runtime environment in which significant_met.R is sourced (or the MetENP package installed via devtools) and the adjustment is executed) — https://github.com/metabolomicsworkbench/MetENP

## Examples

```
sig <- significant_met(met, met_col, "ttest", metadata, normalization, factor1, factor2, factor_col, sample_col, p_adjust="BH")
```

## Evaluation signals

- significant_metabolites.csv exists, parses as a valid CSV, and contains the required fields: metabolite identifier, fold change, log2 fold change, p-value, and p_adjust-adjusted p-value.
- Every adjusted p-value is greater than or equal to its corresponding raw p-value (a basic invariant of holm, hochberg, hommel, and BH adjustments).
- The number of tests underlying the adjustment equals the number of metabolites retained after the chosen NA filter (half_of_min, remove_NAs, or 50percent), not the raw table row count.
- Adjusted p-values are non-negative and bounded by 1, and the adjustment method recorded in the call matches the adjustment column semantics in the output.
- Significantly altered metabolites retained at the chosen threshold are consistent with the fold-change/log2-fold-change direction used in the downstream enrichment and pathway steps.

## Limitations

- Only four adjustment methods (holm, hochberg, hommel, BH) are exposed by significant_met; other R p.adjust methods are not offered.
- The two-group t-test path assumes exactly two levels of factor_col; multi-group designs need the anova_ana function.
- The 50percent NA filter discards metabolites with more than 50% missing values, so sparse metabolite tables can shrink substantially before adjustment.
- half_of_min imputes NAs with half the minimum value, which can distort within-group variance and hence the t-test p-values being adjusted.

## Evidence
