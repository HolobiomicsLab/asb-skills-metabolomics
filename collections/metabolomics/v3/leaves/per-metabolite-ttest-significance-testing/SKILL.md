---
name: per-metabolite-ttest-significance-testing
description: Use when a metabolomics abundance table (e.g., from Metabolomics Workbench)
  must be reduced to significantly altered metabolites between two groups before enrichment
  scoring.
when_to_use_negative:
- The comparison involves more than two groups — MetENP provides a separate anova_ana.R
  function for multi-group analysis rather than the two-factor t-test path
- The goal is already metabolite-class or pathway enrichment scoring (metclassenrichment.R,
  path_enrichmentscore.R) — this skill is the upstream per-metabolite significance
  step, not the enrichment itself
- The input table lacks metadata mapping samples to the two comparison groups (factor_col/sample_col),
  since group assignment is required for the t-test
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: MetENP
  role: Provides the significant_met.R function that implements per-metabolite t-testing,
    NA handling, fold-change computation, and p-value adjustment; loaded via source('path/significant_met.R')
    when not installed as a package
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Runtime environment in which the significant_met function and its dependencies
    (dplyr, tidyr, plyr, etc.) execute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:39a3ff7cda68fe2d2b64370df1e6007ceae22f89be3054e5368308dcd60d10a2
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - MetENP
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# per-metabolite-ttest-significance-testing

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Per-metabolite significance testing in MetENP: compute Student's t-tests across two experimental groups for each metabolite, returning fold change, log2 fold change, p-value, and p-value-adjusted (holm, hochberg, hommel, or BH) significance calls with explicit NA-handling.

## When to use

Use when a metabolomics abundance table (e.g., from Metabolomics Workbench) must be reduced to significantly altered metabolites between two groups before enrichment scoring.

## When NOT to use

- The comparison involves more than two groups — MetENP provides a separate anova_ana.R function for multi-group analysis rather than the two-factor t-test path
- The goal is already metabolite-class or pathway enrichment scoring (metclassenrichment.R, path_enrichmentscore.R) — this skill is the upstream per-metabolite significance step, not the enrichment itself
- The input table lacks metadata mapping samples to the two comparison groups (factor_col/sample_col), since group assignment is required for the t-test

## Inputs

- Metabolite abundance table with metabolite name column and per-sample intensity columns (e.g., ST000915 from Metabolomics Workbench)
- Metadata table mapping samples to factor_col group labels (factor1 vs factor2) and sample_col identifiers
- Parameters: analysis_type (Student's t-test), normalization, NA-handling method (half_of_min | remove_NAs | 50percent), p_adjust method (holm | hochberg | hommel | BH)

## Outputs

- Per-metabolite results table saved as significant_metabolites.csv containing metabolite identifier, fold change, log2 fold change, raw p-value, and adjusted p-value
- Set of significantly altered metabolites feeding downstream metclassenrichment and path_enrichmentscore steps

## How to apply

Retrieve the metabolite abundance table and metadata with MetENP's getmwstudies(studyid, typedata) (e.g., study ST000915), then call significant_met(metabolomics_data, met_col, analysis_type, metadata, normalization, factor1, factor2, factor_col, sample_col, p_adjust) with analysis_type set to the Student's t-test and factor1/factor2 drawn from factor_col. Choose an NA-handling variant before testing: half_of_min (replace NAs with half the minimum value), remove_NAs (drop metabolites with NAs), or 50percent (drop metabolites with >50% NAs). For each metabolite, compute fold change and log2 fold change between factor1 and factor2 alongside the raw p-value and the p_adjust-adjusted p-value, retaining significantly altered metabolites for downstream enrichment (metclassenrichment, path_enrichmentscore). The adjustment method must be one of holm, hochberg, hommel, or BH/fdr.

## Related tools

- **MetENP** (Provides the significant_met.R function that implements per-metabolite t-testing, NA handling, fold-change computation, and p-value adjustment; loaded via source('path/significant_met.R') when not installed as a package) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Runtime environment in which the significant_met function and its dependencies (dplyr, tidyr, plyr, etc.) execute)

## Examples

```
significant_met(metabolomics_data, met_col="metabolite", analysis_type="ttest", metadata, normalization="none", factor1="group1", factor2="group2", factor_col="factor", sample_col="sample", p_adjust="BH")
```

## Evaluation signals

- significant_metabolites.csv exists, is a valid CSV, and contains the required fields: metabolite identifier, fold change, log2 fold change, p-value, and the p_adjust-adjusted p-value
- Adjusted p-values are consistent with the selected method (holm, hochberg, hommel, or BH) applied to the raw p-value column
- Internal consistency: log2 fold change equals log2(fold change) for each metabolite row
- NA-handling invariant matches the chosen variant: rows replaced by half_of_min, removed by remove_NAs, or metabolites with >50% missing values excluded under 50percent
- Downstream enrichment steps (metclassenrichment, path_enrichmentscore) accept the retained significant-metabolite list without missing identifiers

## Limitations

- No statistical validation metrics or quantitative benchmark results are reported for the case studies (Human, bacteria, dog); significance calls were not independently benchmarked
- NA handling materially changes results: half_of_min imputes half the minimum value, which can distort fold changes versus remove_NAs or 50percent filtering
- The two-group t-test path applies only to a single factor contrast (factor1 vs factor2); other designs require the separate anova_ana function

## Evidence
