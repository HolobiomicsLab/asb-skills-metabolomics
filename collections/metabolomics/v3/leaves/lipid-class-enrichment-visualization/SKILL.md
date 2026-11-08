---
name: lipid-class-enrichment-visualization
description: Use when a differential-expression table of lipid species (with p-values)
  is available and the goal is to identify which lipid classes or lipid sets are enriched
  among up- or down-regulated species, and to visualize those enrichments per class.
when_to_use_negative:
- The input is raw lipidomics intensity data that has not yet passed through LipidSigR's
  Differential Expression workflow — no ranked lipid list exists for LSEA
- The analysis goal is pathway-level network or GATOM analysis — the Network workflow
  (two-group data only) is the appropriate choice, not LSEA
- Multi-group data requiring workflows LipidSigR restricts to two-group designs, such
  as Network or Machine learning
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: LipidSigR
  role: Provides enrichment_lsea to run LSEA ranked by p-value and plot_enrichment_lsea
    to draw per-class bar plots and enrichment plots
  repo: https://github.com/BioinfOMICS/LipidSigR
- name: R
  role: Runtime environment (R >= 4.3.0) for loading differential-expression output
    and calling LipidSigR functions
- name: dplyr
  role: Used to manipulate/filter the LSEA results table when selecting a lipid class
    such as Cer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf047
    title: 'LipidSigR: a R-based solution for integrated lipidomics data analysis
      and visualization'
  source_content_hash: sha256:a069e399786e94fd91384538e243928286e47ce0f909f64ec0c61986472fcabc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfOMICS/LipidSigR
  tools:
  - LipidSigR
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# lipid-class-enrichment-visualization

## Summary

Perform Lipid Set Enrichment Analysis (LSEA) on differential-expression results from LipidSigR by ranking lipids by p-value, and generate per-lipid-class bar plots and enrichment plots for the enriched lipid sets. This operationalizes the Enrichment workflow of LipidSigR, the R companion to the LipidSig web platform.

## When to use

Use when a differential-expression table of lipid species (with p-values) is available and the goal is to identify which lipid classes or lipid sets are enriched among up- or down-regulated species, and to visualize those enrichments per class.

## When NOT to use

- The input is raw lipidomics intensity data that has not yet passed through LipidSigR's Differential Expression workflow — no ranked lipid list exists for LSEA
- The analysis goal is pathway-level network or GATOM analysis — the Network workflow (two-group data only) is the appropriate choice, not LSEA
- Multi-group data requiring workflows LipidSigR restricts to two-group designs, such as Network or Machine learning

## Inputs

- Differential-expression output table from LipidSigR's de_analysis step (lipid species with p-values), loaded in R
- Lipidomics data analyzed via a SummarizedExperiment object constructed with LipidSigR tool functions

## Outputs

- LSEA results table of enriched lipid sets by lipid class (e.g., Cer, TAG, PC O-)
- lsea_bar_plot_Cer.png — LSEA bar plot of enriched lipid sets for the selected lipid class
- lsea_enrichment_plot_Cer.png — enrichment plot for the selected lipid class

## How to apply

Load the differential-expression output table produced by LipidSigR's upstream Differential Expression workflow in R, then call enrichment_lsea with rank_by='pval' so lipids are ranked by their differential-expression p-values for LSEA. Inspect the LSEA output table to identify enriched lipid sets grouped by lipid class, and select the class of interest (e.g., ceramide (Cer)). Call plot_enrichment_lsea twice: once to draw the LSEA bar plot of enriched lipid sets for the selected class, and once to draw the enrichment plot, saving each as a PNG. Validate that both PNG files exist and are non-empty and that plotted results correspond to the selected class's entries in the LSEA output table. LSEA is one of two approaches in the Enrichment workflow, alongside Over Representation Analysis (ORA).

## Related tools

- **LipidSigR** (Provides enrichment_lsea to run LSEA ranked by p-value and plot_enrichment_lsea to draw per-class bar plots and enrichment plots) — https://github.com/BioinfOMICS/LipidSigR
- **R** (Runtime environment (R >= 4.3.0) for loading differential-expression output and calling LipidSigR functions)
- **dplyr** (Used to manipulate/filter the LSEA results table when selecting a lipid class such as Cer)

## Examples

```
lsea_res <- enrichment_lsea(de_result, rank_by = 'pval'); plot_enrichment_lsea(lsea_res, lipid_class = 'Cer', plot_type = 'bar', out_name = 'lsea_bar_plot_Cer.png')
```

## Evaluation signals

- Both lsea_bar_plot_Cer.png and lsea_enrichment_plot_Cer.png exist on disk and are non-empty
- Plotted enrichment results correspond to the Cer class entries in the LSEA output table
- Lipid ranking in enrichment_lsea used rank_by='pval' from the upstream differential-expression p-values
- Enriched classes match expected biology (e.g., in the HFrEF case, up-regulated species enriched in TAGs, down-regulated in PC O-)
- Input to enrichment_lsea is the de_analysis output table, confirming correct workflow chaining

## Limitations

- Only two-group data can conduct the Network and Machine learning workflows; Enrichment outputs should be interpreted in the context of the case-versus-control design LipidSigR demonstrates
- Requires R 4.3.0 or later; on Windows, LipidSigR is supported only on Windows 10 or later
- LipidSigR depends on several packages (e.g., fgsea, gatom, mixOmics) that may need manual installation if dependencies fail

## Evidence

- [readme] Enrichment workflow: ORA and LSEA approaches: "'Over Representation Analysis (ORA)' and 'Lipid Set Enrichment Analysis (LSEA)' to illustrates significant lipid species enriched in the categories of lipid class"
- [readme] LipidSigR is an R package based on LipidSig: ""LipidSigR" is an R package developed based on LipidSig web-based tool"
- [methods] Enrichment workflow within six main workflows: "We provide 6 main analysis workflows, which is "Profiling""
- [readme] Package integrates comprehensive lipidomics data mining: "This package integrates a comprehensive analysis for streamlined data mining of 
lipidomic datasets."
- [results] LSEA applied to differential expression results enriches TAGs and PC O-: [evidence span withheld — anchor status: not_found]
