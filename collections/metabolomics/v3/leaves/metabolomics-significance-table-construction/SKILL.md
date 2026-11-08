---
name: metabolomics-significance-table-construction
description: Use when starting from a metabolite names/concentration matrix with sample
  metadata and you need to determine which metabolites are significantly altered between
  comparison groups before class or pathway enrichment.
when_to_use_negative:
- The input is already a curated list of significant metabolites with directions —
  skip directly to metclassenrichment or path_enrichmentscore.
- The goal is pathway-level enrichment without a metabolite-class or significance
  step — significant_met alone is insufficient; mapping to KEGG pathways (map_keggid,
  met_pathways) is a separate skill.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetENP
  role: R package providing convert_refmet, anova_ana, significant_met, and plot_volcano
    used to build the significance table
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: RefMet
  role: Standard metabolite nomenclature used to normalize metabolite names before
    significance testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:f3a7bc155e56175c7ba5b8718807835fac2ea8d641f68492966643be1a0979ba
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - MetENP
  - R
  - RefMet
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# metabolomics-significance-table-construction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Construct the significant-metabolite table that MetENP's downstream enrichment steps (metclassenrichment, path_enrichmentscore) consume: metabolite names/concentrations and metadata are converted to RefMet standard nomenclature (convert_refmet), statistical testing via ANOVA (anova_ana) yields fold changes and p-values, and significant_met applies user-specified fold-change and p-value cutoffs to flag metabolites as significantly increased or decreased.

## When to use

Use when starting from a metabolite names/concentration matrix with sample metadata and you need to determine which metabolites are significantly altered between comparison groups before class or pathway enrichment.

## When NOT to use

- The input is already a curated list of significant metabolites with directions — skip directly to metclassenrichment or path_enrichmentscore.
- The goal is pathway-level enrichment without a metabolite-class or significance step — significant_met alone is insufficient; mapping to KEGG pathways (map_keggid, met_pathways) is a separate skill.

## Inputs

- Metabolite names (or names plus concentrations) data frame from a Metabolomics Workbench study or custom dataset
- Sample metadata (group/class information for comparisons)
- User-specified cutoffs: fold-change threshold, p-value threshold, and optional chemical-class filter
- RefMet standard nomenclature mapping (via convert_refmet)

## Outputs

- Significant-metabolite table (R data frame / CSV) with per-metabolite fold change, p-value, direction (increase/decrease), and RefMet class
- Volcano plot of significantly altered metabolites (plot_volcano)

## How to apply

Load the metabolite information (names, or names and concentrations, plus metadata) and first standardize metabolite names via convert_refmet, which maps them to RefMet standard nomenclature so counts are comparable across experiments. Run anova_ana to compute per-metabolite statistics across the comparison groups (fold changes and p-values), which also supports volcano-plot inspection of the altered metabolites. Then call significant_met, supplying user-specified cutoffs for fold change and p-value (and optionally chemical class and analysis type) to label each metabolite as significantly increased, decreased, or not significant. The rationale is that enrichment scores in metclassenrichment are only meaningful if computed on a defensible significance table: the counts of significant metabolites (L) and of significant metabolites per RefMet class (M) feed the hypergeometric test phyper(M-1, L, N-L, k, lower.tail=FALSE), where N is all metabolites detected in the study and k is all metabolites in a class.

## Related tools

- **MetENP** (R package providing convert_refmet, anova_ana, significant_met, and plot_volcano used to build the significance table) — https://github.com/metabolomicsworkbench/MetENP
- **RefMet** (Standard metabolite nomenclature used to normalize metabolite names before significance testing)

## Examples

```
source('path/convert_refmet.R'); source('path/anova_ana.R'); source('path/significant_met.R'); sig <- significant_met(met_conc_data, metadata, foldchange_cutoff, pvalue_cutoff)
```

## Evaluation signals

- Every metabolite row carries a fold change and p-value from anova_ana, and a direction label (increased/decreased/not significant) consistent with the user-specified cutoffs.
- All metabolite names in the significance table resolve to RefMet standard nomenclature (convert_refmet output has no unmatched names or unmatched rows are explicitly flagged).
- Row count of significant metabolites (L) plus the per-class significant counts (M) are internally consistent: sum of M across classes must not exceed L, and N equals all metabolites detected in the study.
- Downstream metclassenrichment on this table produces p-values within [0, 1] and a valid enrichment score per RefMet class.
- Volcano plot (plot_volcano) visually agrees with the tabulated significant set at the chosen fold-change and p-value cutoffs.

## Limitations

- Cutoff choices (fold change, p-value) are user-specified and strongly affect which metabolites count as significant; the article's case studies used different parameters per species with no quantitative benchmark reported.
- Requires metabolite names to be mappable to RefMet standard nomenclature; unmappable names reduce usable counts of N, L, k, and M in the enrichment test.
- Statistical testing assumes appropriate group structure in the metadata (ANOVA over comparison groups); with custom datasets the user must supply concentrations and metadata correctly.

## Evidence

- [methods] significant metabolites within that class and phyper hypergeometric test: [evidence span withheld — anchor status: not_found]
