---
name: fold-change-log2fc-calculation
description: Use when comparing metabolite abundance/intensity between two groups
  (factor1 vs factor2 from a metadata factor column) and per-metabolite fold change,
  log2FC, and significance values are needed before enrichment analysis.
when_to_use_negative:
- When more than two experimental groups must be compared simultaneously — use MetENP's
  anova_ana for multi-group ANOVA instead of the two-group t-test fold-change workflow
- When the goal is enrichment scoring of metabolite classes or KEGG pathways rather
  than identifying significantly altered metabolites — fold change calculation is
  only the upstream step
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MetENP
  role: Provides significant_met(), which computes Student's t-test per metabolite,
    fold change, log2 fold change, and adjusted p-values with three NA-handling variants
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Runtime environment; significant_met.R is sourced from the package or directly
    via source('path/significant_met.R')
- name: MetENPweb
  role: Web-application alternative on Metabolomics Workbench for running the same
    significant-metabolite calculation through a GUI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:d256a120c01f331d3e465d591b52047dc436b219296f9ea38b3dfc4f211e3a41
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - MetENP
  - MetENPweb
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fold-change-log2fc-calculation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Per-metabolite fold change and log2 fold change computation between two experimental groups as performed by MetENP's significant_met function, alongside raw and adjusted p-values from a chosen statistical test.

## When to use

Use when comparing metabolite abundance/intensity between two groups (factor1 vs factor2 from a metadata factor column) and per-metabolite fold change, log2FC, and significance values are needed before enrichment analysis.

## When NOT to use

- When more than two experimental groups must be compared simultaneously — use MetENP's anova_ana for multi-group ANOVA instead of the two-group t-test fold-change workflow
- When the goal is enrichment scoring of metabolite classes or KEGG pathways rather than identifying significantly altered metabolites — fold change calculation is only the upstream step

## Inputs

- Metabolite abundance/intensity table from Metabolomics Workbench (e.g. ST000915) via getmwstudies(studyid, typedata)
- Sample metadata with factor_col, sample_col, and the two comparison groups (factor1, factor2)
- NA-handling method choice: half_of_min, remove_NAs, or 50percent
- p-value adjustment method: holm, hochberg, hommel, or BH

## Outputs

- significant_metabolites.csv containing metabolite identifier, fold change, log2 fold change, p-value, and adjusted p-value
- Retained set of significantly altered metabolites for downstream enrichment (metclassenrichment, path_enrichmentscore)

## How to apply

Retrieve the metabolite abundance table and metadata (e.g. via MetENP's getmwstudies on a Metabolomics Workbench study such as ST000915), then call significant_met with analysis_type set to Student's t-test, factor1/factor2 drawn from factor_col, and a p_adjust method among holm, hochberg, hommel, or BH/fdr. Before testing, handle missing intensities with one of three NA-filtering variants: half_of_min (replace NAs with half the minimum value), remove_NAs (drop metabolites with NAs), or 50percent (drop metabolites with >50% NAs). For each metabolite, compute fold change and log2 fold change between the two groups together with the raw p-value and the p_adjust-adjusted p-value, and retain the significantly altered metabolites. Write the per-metabolite table as significant_metabolites.csv with fields: metabolite identifier, fold change, log2 fold change, p-value, and adjusted p-value.

## Related tools

- **MetENP** (Provides significant_met(), which computes Student's t-test per metabolite, fold change, log2 fold change, and adjusted p-values with three NA-handling variants) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Runtime environment; significant_met.R is sourced from the package or directly via source('path/significant_met.R'))
- **MetENPweb** (Web-application alternative on Metabolomics Workbench for running the same significant-metabolite calculation through a GUI)

## Examples

```
source('path/significant_met.R'); sig <- significant_met(metabolomics_data, met_col, analysis_type, metadata, normalization, factor1, factor2, factor_col, sample_col, p_adjust)
```

## Evaluation signals

- significant_metabolites.csv exists and parses as a valid CSV
- Output contains the required fields: metabolite identifier, fold change, log2 fold change, p-value, and p_adjust-adjusted p-value for the selected method (holm/hochberg/hommel/BH)
- log2 fold change values are consistent with fold change values (log2FC = log2(FC)) for each metabolite row
- NA-handling variant applied matches the chosen option (e.g. 50percent removes metabolites with >50% missing values; half_of_min imputes half the column minimum)
- Number of retained significantly altered metabolites is consistent with the adjusted p-value threshold for the chosen p_adjust method

## Limitations

- NA handling choices materially change results: half_of_min imputes half the minimum value, remove_NAs drops metabolites with any NA, and 50percent drops metabolites with >50% NAs — the variant must be reported with results
- The default analysis_type is a two-group Student's t-test; it does not cover multi-group designs (use anova_ana for those)
- p-value adjustment options are limited to holm, hochberg, hommel, and BH/fdr

## Evidence

- [other] MetENP's significant_met function computes per-metabolite Student's t-tests over chosen experimental groups: [evidence span withheld — anchor status: not_found]
- [other] NA handling methods: [evidence span withheld — anchor status: not_found]
- [discussion] workflow step b: "b) calculate significantly altered metabolites"
- [readme] README source step: "source('path/significant_met.R')"
- [readme] package purpose: "MetENP is a R package that enables detection of significant metabolites from metabolite information (names or names and concentration along with metadata information)"
