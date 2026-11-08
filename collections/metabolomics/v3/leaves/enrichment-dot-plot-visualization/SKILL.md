---
name: enrichment-dot-plot-visualization
description: Use when you have a ranked pathway enrichment table (Fisher's Exact Test
  with Benjamini-Hochberg adjusted p-values) and need a dot plot that highlights the
  top significantly enriched pathways for figures or validation.
when_to_use_negative:
- Input is a raw metabolomics intensity matrix — run differential analysis (run_de()
  with fc_threshold = 1, pval_threshold = 0.05) and enrichment first; there are no
  adjusted p-values to plot yet
- The goal is ranking metabolite centrality or network topology rather than pathway
  significance — the RBC plot or network graph modules are the appropriate visualizations
- Results come from rank-based MetSEA alone and NES-based ordering is wanted — use
  the MetSEA/gsea plot rather than the over-representation dot plot
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: EnrichMet
  role: R package that generates the dot plots of -log10(adjusted p-values), pathway
    impact plots, and MetSEA plots from the enrichment results in a single function
    call
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Execution environment for running EnrichMet and rendering/loving the ggplot/ggraph/ComplexHeatmap
    plot objects
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:cf7bfbc99443fd0e2b8cd16ca4008c9841476f980a8f9f13f5f1a7da9f206be5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# enrichment-dot-plot-visualization

## Summary

Visualization skill for rendering pathway over-representation and enrichment results as dot plots of -log10(adjusted p-values), as used by the EnrichMet R package to highlight the top significantly enriched pathways (e.g., Figure 2A/2B and the lipidomics Figure S2).

## When to use

Use when you have a ranked pathway enrichment table (Fisher's Exact Test with Benjamini-Hochberg adjusted p-values) and need a dot plot that highlights the top significantly enriched pathways for figures or validation.

## When NOT to use

- Input is a raw metabolomics intensity matrix — run differential analysis (run_de() with fc_threshold = 1, pval_threshold = 0.05) and enrichment first; there are no adjusted p-values to plot yet
- The goal is ranking metabolite centrality or network topology rather than pathway significance — the RBC plot or network graph modules are the appropriate visualizations
- Results come from rank-based MetSEA alone and NES-based ordering is wanted — use the MetSEA/gsea plot rather than the over-representation dot plot

## Inputs

- Pathway enrichment results table with Pathway, P_value, Adjusted_P_value, and Enrichment_Ratio columns (S3 data.frame from enrichmet())
- Ranked pathway list from Fisher's Exact Test over-representation analysis (e.g., ora_pathway_enrichment_table.csv)
- Optional: GSEA/MetSEA results with pathway, pval, padj, NES columns
- Optional: pathway impact scores (betweenness-centrality based) for impact plots

## Outputs

- Dot plot of -log10(adjusted p-values) highlighting top significantly enriched pathways (e.g., figure2a_ora_dotplot.png)
- Pathway enrichment plot (results$pathway_plot)
- Pathway impact plot: significance vs. topology-based impact score (results$impact_plot)
- MetSEA plot of -log10(adjusted p-values) for fgsea results (results$gsea_plot)

## How to apply

After running over-representation analysis (Fisher's Exact Test on 2x2 contingency tables per pathway) and BH adjustment, take the ranked table ordered by adjusted p-value and visualize it as a dot plot of -log10(adjusted p-values), highlighting pathways passing the adjusted p < 0.05 significance threshold. In EnrichMet, the `enrichmet()` function generates these plots automatically (e.g., `pathway_plot` for Fisher's exact test enrichment and `impact_plot` combining significance with the topology-based pathway impact score); use `top_n` to control how many pathways are displayed. Dedicated plot objects are returned as S3/S4 plot objects, so users can render individual plots without rerunning the full workflow. Validate that the plot ordering matches the ranked enrichment table and the published figure (pyrimidine metabolism as top pathway in the control-vs-mutant lung cancer case study).

## Related tools

- **EnrichMet** (R package that generates the dot plots of -log10(adjusted p-values), pathway impact plots, and MetSEA plots from the enrichment results in a single function call) — https://github.com/biodatalab/enrichmet.git
- **R** (Execution environment for running EnrichMet and rendering/loving the ggplot/ggraph/ComplexHeatmap plot objects)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, top_n = 10); results$pathway_plot; results$impact_plot
```

## Evaluation signals

- Plot ordering matches the ranked enrichment table sorted by adjusted p-value, with pyrimidine metabolism as the top pathway in the control-vs-mutant lung cancer case study (Figure 2A validation)
- Only pathways with adjusted p-value < 0.05 are highlighted as significantly enriched
- Returned results contain the expected plot objects (pathway_plot, impact_plot, gsea_plot, etc.) with ggplot/ggraph/Heatmap classes
- Number of displayed pathways respects the top_n parameter and pathway_enrichment_results is limited to p <= cutoff entries
- For the lipidomics extension, the dot plot reproduces the Figure S2 output with enriched LION pathways identified at adjusted p < 0.05

## Limitations

- Only the top_n pathways are plotted, so low-ranked pathways may be omitted from the visualization
- Visualization depends on Benjamini-Hochberg adjusted p-values; if few pathways pass p <= cutoff, few dots appear (README example reports only 10 of 32 tested pathways reported at cutoff)
- LIPID ontology content is not shipped with the package — lipidomics dot plots require fetching LION content at runtime via BioPortal/LION interface, which can fail if the API is unavailable

## Evidence

- [intro] visualized using dot plots of -log10(adjusted p-values), highlighting the top significantly enriched pathways: "visualized using dot plots of -log10(adjusted p-values), highlighting the top significantly enriched pathways"
- [readme] Pathway impact: significance vs. topology-based impact score: [evidence span withheld — anchor status: external_doc]
- [readme] For users who wish to run only a specific analysis or generate selected plots rather than executing the full workflow: "For users who wish to run only a specific analysis or generate selected plots rather than executing the full workflow"
- [intro] A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched: "A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched"
