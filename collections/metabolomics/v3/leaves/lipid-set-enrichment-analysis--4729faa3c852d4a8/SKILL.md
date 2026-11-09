---
name: lipid-set-enrichment-analysis
description: Use when you have a LipidSigR differential-expression output table of
  lipid species and want to test whether a priori-defined lipid sets (e.g., lipid
  classes like Cer, TAG, PC O-) show statistically significant, concordant differences
  between two biological states (e.g., phenotypes).
when_to_use_negative:
- The analysis goal is testing over-representation of a pre-selected significant lipid
  list rather than a ranked full lipidome — use LipidSigR's ORA approach instead
- The input is raw lipidomics intensity data that has not yet been processed through
  the Differential Expression workflow — LSEA requires differential-expression results
  as input
- 'The data are multi-group: enrichment interpretation in the surfaced case study
  is case-versus-control, and Network/Machine-learning-style two-group constraints
  signal this package''s two-group orientation'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0091
tools:
- name: LipidSigR
  role: Provides enrichment_lsea (LSEA on differential-expression results ranked by
    p-value) and plot_enrichment_lsea (bar plot and enrichment plot per lipid class)
  repo: https://github.com/BioinfOMICS/LipidSigR
- name: R
  role: Runtime environment; LipidSigR requires R 4.3.0 or later
- name: dplyr
  role: Used in R to load and manipulate the differential-expression output table
    before calling LSEA functions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf047
    title: 'LipidSigR: a R-based solution for integrated lipidomics data analysis
      and visualization'
  source_content_hash: sha256:0c9ec30179bcf593ab42b4f185c13f3766403e6e4b3ce07d0f3a4c2f99ace1b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfOMICS/LipidSigR
  tools:
  - LipidSigR
  - R
  - dplyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lipid-set-enrichment-analysis

## Summary

Run Lipid Set Enrichment Analysis (LSEA) with LipidSigR to rank differential-expression results (e.g., by p-value) and identify lipid classes — such as TAGs, PC O-, or Ceramides — enriched among up- or down-regulated lipid species, then visualize the results with dedicated LSEA plots.

## When to use

Use when you have a LipidSigR differential-expression output table of lipid species and want to test whether a priori-defined lipid sets (e.g., lipid classes like Cer, TAG, PC O-) show statistically significant, concordant differences between two biological states (e.g., phenotypes).

## When NOT to use

- The analysis goal is testing over-representation of a pre-selected significant lipid list rather than a ranked full lipidome — use LipidSigR's ORA approach instead
- The input is raw lipidomics intensity data that has not yet been processed through the Differential Expression workflow — LSEA requires differential-expression results as input
- The data are multi-group: enrichment interpretation in the surfaced case study is case-versus-control, and Network/Machine-learning-style two-group constraints signal this package's two-group orientation

## Inputs

- Differential-expression output table from LipidSigR's de_analysis step (lipid species with p-values)
- SummarizedExperiment object constructed via LipidSigR tool functions (upstream requirement)

## Outputs

- LSEA results table of enriched lipid sets per lipid class
- LSEA bar plot of enriched lipid sets for a selected lipid class (e.g., lsea_bar_plot_Cer.png)
- LSEA enrichment plot for the selected lipid class (e.g., lsea_enrichment_plot_Cer.png)

## How to apply

Load the differential-expression table produced by LipidSigR's 'Differential Expression' workflow and call the enrichment_lsea function with rank_by='pval' so lipids are ranked by their differential-expression p-values. LSEA is one of the two main approaches in LipidSigR's Enrichment workflow (alongside ORA) and determines whether predefined lipid sets, categorized by lipid class, are enriched among significantly changed species. Select a lipid class of interest from the LSEA results (e.g., Cer) and call plot_enrichment_lsea to draw both the LSEA bar plot of enriched lipid sets and the enrichment plot, saving each as a PNG. Validate by confirming the PNG files exist and are non-empty, and that plotted enrichment results correspond to the Cer entries in the LSEA output table.

## Related tools

- **LipidSigR** (Provides enrichment_lsea (LSEA on differential-expression results ranked by p-value) and plot_enrichment_lsea (bar plot and enrichment plot per lipid class)) — https://github.com/BioinfOMICS/LipidSigR
- **R** (Runtime environment; LipidSigR requires R 4.3.0 or later)
- **dplyr** (Used in R to load and manipulate the differential-expression output table before calling LSEA functions)

## Examples

```
lsea_result <- enrichment_lsea(de_result, rank_by = 'pval'); plot_enrichment_lsea(lsea_result, lipid_class = 'Cer')
```

## Evaluation signals

- Both output PNGs (e.g., lsea_bar_plot_Cer.png and lsea_enrichment_plot_Cer.png) exist and are non-empty
- Plotted enrichment results correspond to the Cer-class entries in the LSEA output table
- Lipid ranking in enrichment_lsea uses rank_by='pval' so ordering matches the differential-expression p-values
- Enriched sets are interpretable in lipid-class categories (e.g., up-regulated species enriched in TAGs, down-regulated in PC O-)

## Limitations

- Only two-group data can conduct some workflows ('Network' and 'Machine learning'); the two-group case-versus-control design is the demonstrated scope for differential-expression-driven LSEA
- LipidSigR requires R 4.3.0 or later; on Windows, only Windows 10 or later is supported
- LSEA in LipidSigR depends on the fgsea package, so missing dependencies must be installed via BiocManager

## Evidence

- [readme] The enrichment workflow provides two main approaches: 'Over Representation Analysis (ORA)' and 'Lipid Set Enrichment Analysis (LSEA)': "The enrichment workflow provides two main approaches: 'Over Representation Analysis (ORA)' and 'Lipid Set Enrichment Analysis (LSEA)'"
- [readme] determine whether an a priori-defined set of lipids shows statistically significant, concordant differences between different biological states (e.g., phenotypes): [evidence span withheld — anchor status: external_doc]
- [intro] We provide five main analysis workflows for analyzing two-group and multi-group data: "We provide five main analysis workflows for analyzing 
two-group and multi-group data:"
- [intro] You need to have R 4.3.0 or a later version installed for running LipidSigR.: "You need to have R 4.3.0 or a later version installed for running LipidSigR."
- [readme] LipidSigR depends on several packages, which can be installed using the below commands: BiocManager::install(c('fgsea', ...)): "BiocManager::install(
    c('fgsea', 'gatom', 'mixOmics', 'S4Vectors', 'BiocGenerics',
      'SummarizedExperiment', 'rgoslin'))"
