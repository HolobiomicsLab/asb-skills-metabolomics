---
name: enrichment-dotplot-visualization
description: Use when you have a MetSEA/fgsea results table with NES, pval and padj
  columns (or Fisher enrichment results with Adjusted_P_value) and need a dot plot
  of -log10(adjusted p-values) to present top enriched pathways.
when_to_use_negative:
- Input is only a binary list of significant metabolite IDs with no fold change or
  p-value ranking statistics — MetSEA/fgsea requires a ranked list, and over-representation
  analysis (Fisher's Exact Test) is the appropriate module instead
- The analysis goal is topology-based pathway impact or betweenness centrality visualization
  rather than enrichment significance (use the pathway impact plot / RBC plot outputs
  instead)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3172
tools:
- name: EnrichMet
  role: R package whose enrichmet() function runs MetSEA via fgsea and generates the
    gsea_plot dot plot of MetSEA results and the pathway_plot/impact_plot visualizations
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Language in which EnrichMet is implemented; used to invoke the package and
    render ggplot/ggraph-based dot plots
- name: KEGG
  role: Source of the curated pathway-metabolite associations converted to GMT format
    that underlie the enrichment results being visualized
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:3218affbc6fac9e74bf92510218a6bb04e817fb2cff9bc42cb61f7dc8156c572
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# enrichment-dotplot-visualization

## Summary

Visualize metabolite set enrichment analysis (MetSEA/fgsea) and Fisher-based pathway enrichment results as dot plots of -log10(adjusted p-values), highlighting the top significantly enriched pathways as done in EnrichMet's lung cancer case study (Figure 2C).

## When to use

Use when you have a MetSEA/fgsea results table with NES, pval and padj columns (or Fisher enrichment results with Adjusted_P_value) and need a dot plot of -log10(adjusted p-values) to present top enriched pathways.

## When NOT to use

- Input is only a binary list of significant metabolite IDs with no fold change or p-value ranking statistics — MetSEA/fgsea requires a ranked list, and over-representation analysis (Fisher's Exact Test) is the appropriate module instead
- The analysis goal is topology-based pathway impact or betweenness centrality visualization rather than enrichment significance (use the pathway impact plot / RBC plot outputs instead)

## Inputs

- fgsea/MetSEA results data.frame with pathway, pval, padj, NES columns (results$gsea_results)
- Fisher pathway enrichment table with Pathway, P_value, Adjusted_P_value, Enrichment_Ratio columns (results$pathway_enrichment_results)
- Ranked metabolite list (log2 fold change and p-values) from differential analysis of control vs mutant metabolomics summary statistics (met_id, pval/padj, log2fc)
- GMT-formatted KEGG pathway-metabolite annotations

## Outputs

- Dot plot of -log10(adjusted p-values) highlighting the top significantly enriched pathways (results$gsea_plot / Figure 2C-style plot)
- Filtered enrichment table of significant pathways (adjusted p < 0.05) with NES and adjusted-p fields
- Optional companion pathway enrichment plot and pathway impact plot (results$pathway_plot, results$impact_plot)

## How to apply

Run the MetSEA module of EnrichMet, which converts KEGG pathway annotations into Gene Matrix Transposed (GMT) format and calls fgsea on metabolites ranked by log2 fold change and p-value from the differential analysis (e.g. run_de with fc_threshold = 1, pval_threshold = 0.05). Take the resulting gsea_results data.frame containing the pathway, pval, padj, and NES columns, and filter pathways to those significant at adjusted p-value < 0.05, which the paper defines as the significance criterion when accompanied by a positive or negative NES. Plot the top pathways as a dot plot with -log10(adjusted p-values) on one axis (the package exposes this directly as results$gsea_plot, alongside pathway_plot and impact_plot for Fisher/impact results). For the lung cancer control vs mutant case study, confirm the recovered top pathways group into the expected categories: metabolic reprogramming and energy production, nutrient uptake and homeostasis (Figure 2C).

## Related tools

- **EnrichMet** (R package whose enrichmet() function runs MetSEA via fgsea and generates the gsea_plot dot plot of MetSEA results and the pathway_plot/impact_plot visualizations) — https://github.com/biodatalab/enrichmet.git
- **R** (Language in which EnrichMet is implemented; used to invoke the package and render ggplot/ggraph-based dot plots)
- **KEGG** (Source of the curated pathway-metabolite associations converted to GMT format that underlie the enrichment results being visualized)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, example_data = example_summary, kegg_lookup = kegg_lookup, reactome_df = reactome_df, top_n = 10); results$gsea_plot
```

## Evaluation signals

- The MetSEA results table contains NES and adjusted-p (padj) columns before plotting
- Only pathways with adjusted p-value < 0.05 are reported as significantly enriched in the figure
- The dot plot encodes -log10(adjusted p-values) and shows only the top pathways (e.g. top_n = 10 in enrichmet())
- Recovered pathway categories match the paper's Figure 2C grouping: metabolic reprogramming and energy production, nutrient uptake and homeostasis
- Plot objects are valid S3/S4 ggplot objects among the named outputs (pathway_plot, impact_plot, gsea_plot, rbc_plot, network_plot, heatmap_plot, membership_plot, interaction_plot)

## Limitations

- Lipid ontology (LION) content is not shipped with the package; it must be fetched at runtime via the BioPortal/LION API, so lipidomics visualizations depend on network access and caching (BiocFileCache)
- Significance depends on the adjusted p-value cutoff (< 0.05); with weak signals no pathways pass the threshold and the dot plot may be empty
- EnrichMet/fgsea results depend on the ranking of metabolites by log2 fold change and p-value; poorly powered differential statistics will change which pathways appear in the plot

## Evidence

- [intro] Visualize the top significantly enriched pathways as dot plots: "visualized using dot plots of -log10(adjusted p-values), highlighting the top significantly enriched pathways"
- [intro] MetSEA significance criterion at adjusted p < 0.05: "A positive or negative NES accompanied by small adjusted p-value (< 0.05) indicates that the corresponding pathway is significantly enriched"
- [readme] MetSEA is performed via fgsea within EnrichMet: "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
- [readme] enrichmet() returns result tables that feed the plots: "The `enrichmet()` function produces three tables (S3 `data.frame` objects): pathway enrichment results, MetSEA results, and metabolite centrality"
