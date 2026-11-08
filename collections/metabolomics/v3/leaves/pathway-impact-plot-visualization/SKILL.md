---
name: pathway-impact-plot-visualization
description: Use when you have ORA enrichment results (adjusted Fisher's exact-test
  p-values on KEGG pathways) plus a significant metabolite list, and need to prioritize
  pathways by network topology impact rather than p-value rank alone.
when_to_use_negative:
- When only a ranked metabolite list is available (no significant-metabolite set for
  contingency-table testing) — MetSEA via fgsea is the appropriate module instead
- When analyzing lipidomics or non-KEGG feature types without a pathway-feature mapping
  — supply a custom Pathway/Metabolites table or use the LION-based lipidomics workflow
  rather than the KEGG impact plot
- When the input is a raw metabolomics intensity matrix — differential analysis (run_de
  with fc_threshold=1, pval_threshold=0.05) must be performed first to obtain significant
  metabolites
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
tools:
- name: EnrichMet
  role: Produces the pathway impact plot via the enrichmet() function's impact_plot
    output, combining centrality and enrichment results
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Implementation language in which the enrichment, centrality computation, and
    impact plot rendering run
- name: KEGG
  role: Source of curated pathway-metabolite annotations used to map significant metabolites
    to pathways
- name: STITCH
  role: Source of chemical-chemical relationships (filtered at combined score >= 100)
    used to build the interaction network for betweenness centrality
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:60dab3e4e12a5ca432555e19d7122c9fcedeee52dfea4f97037ac61691ebf1d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  - STITCH
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# pathway-impact-plot-visualization

## Summary

Renders the EnrichMet pathway impact plot, which combines -log10(BH-adjusted p-values) from Fisher's exact-test over-representation analysis with a topology-based pathway impact score derived from betweenness centrality on a STITCH metabolite-metabolite interaction network. This surfaces pathways (e.g., caffeine metabolism) that statistical significance alone would rank below top hits (e.g., pyrimidine metabolism).

## When to use

Use when you have ORA enrichment results (adjusted Fisher's exact-test p-values on KEGG pathways) plus a significant metabolite list, and need to prioritize pathways by network topology impact rather than p-value rank alone.

## When NOT to use

- When only a ranked metabolite list is available (no significant-metabolite set for contingency-table testing) — MetSEA via fgsea is the appropriate module instead
- When analyzing lipidomics or non-KEGG feature types without a pathway-feature mapping — supply a custom Pathway/Metabolites table or use the LION-based lipidomics workflow rather than the KEGG impact plot
- When the input is a raw metabolomics intensity matrix — differential analysis (run_de with fc_threshold=1, pval_threshold=0.05) must be performed first to obtain significant metabolites

## Inputs

- ORA pathway enrichment results with P_value and Adjusted_P_value per KEGG pathway (Fisher's Exact Test, Benjamini-Hochberg adjusted)
- List of significant metabolites (absolute fold change >= 1 and adjusted p-value < 0.05), ideally as KEGG compound IDs
- STITCH chemical-chemical interaction data for building the metabolite-metabolite network
- Pathway-vs-metabolite mapping table (e.g., KEGG hsa pathways fetched via fetch_kegg_pathway_metabolites)

## Outputs

- Pathway impact plot (impact_plot object) plotting -log10(adjusted p-value) versus topology-based pathway impact score with top significant pathways highlighted
- Per-pathway impact scores derived from betweenness centrality on the STITCH interaction network
- Metabolite betweenness centrality table (RBC_Metabolite per metabolite)

## How to apply

First run over-representation analysis with Fisher's Exact Test on a 2x2 contingency table using significant metabolites selected at absolute fold change >= 1 and adjusted p value < 0.05, with Benjamini-Hochberg adjustment. Next, construct the metabolite-metabolite interaction network from STITCH chemical-chemical relationships, filtering interactions at combined score >= 100 to ensure biological relevance. Compute betweenness centrality on this network and derive the pathway impact score, which quantifies the extent to which a pathway contributes to overall network connectivity and information flow. Finally, combine the impact scores with -log10(adjusted p-values) and render the pathway impact plot, highlighting the top significantly enriched pathways. Validation for the control vs mutant lung cancer case study: confirm both pyrimidine metabolism (top adjusted p-value) and caffeine metabolism (strong structural impact) appear among the high-impact significant pathways, matching the reported Figure 2B pattern.

## Related tools

- **EnrichMet** (Produces the pathway impact plot via the enrichmet() function's impact_plot output, combining centrality and enrichment results) — https://github.com/biodatalab/enrichmet.git
- **R** (Implementation language in which the enrichment, centrality computation, and impact plot rendering run)
- **KEGG** (Source of curated pathway-metabolite annotations used to map significant metabolites to pathways)
- **STITCH** (Source of chemical-chemical relationships (filtered at combined score >= 100) used to build the interaction network for betweenness centrality)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, kegg_lookup = kegg_lookup, reactome_df = reactome_df); results$impact_plot
```

## Evaluation signals

- In the control vs mutant lung cancer case study, both pyrimidine metabolism (top adjusted p-value) and caffeine metabolism (strong structural impact) must appear among high-impact significant pathways, reproducing the Figure 2B pattern
- The network must be built only from STITCH interactions with combined score >= 100 before betweenness centrality is computed
- ORA p-values must be Benjamini-Hochberg adjusted Fisher's Exact Test results on a 2x2 contingency table, and the plot's y-axis must be -log10(adjusted p-value)
- The rendered object must be the impact_plot output (a plot object) from the enrichmet() results list
- Only significant metabolites meeting absolute fold change >= 1 and adjusted p value < 0.05 should feed the contingency table

## Limitations

- The impact score depends on the STITCH network and the combined score >= 100 filtering threshold, so centrality values shift if interaction data or cutoffs change
- Pathway and compound annotations are fetched at runtime from KEGG REST (and Reactome mappings) and cached with BiocFileCache; network issues or API changes affect reproducibility, and the package does not redistribute database dumps
- The enrichment background should ideally be restricted to measured metabolites via backgroundMetabolites to reduce bias from database compounds never observed

## Evidence

- [intro] Pathway impact score definition: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [intro] STITCH network filter: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [intro] Impact plot rendering: "visualized using dot plots of -log10(adjusted p-values), highlighting the top significantly enriched pathways"
- [readme] Impact plot in README: [evidence span withheld — anchor status: external_doc]
- [readme] Centrality computation in package: "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
