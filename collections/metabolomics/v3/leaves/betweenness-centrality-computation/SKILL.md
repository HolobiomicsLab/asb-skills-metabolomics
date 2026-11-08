---
name: betweenness-centrality-computation
description: Use when a metabolomics differential analysis has yielded a significant
  metabolite list (absolute fold change ≥ 1, adjusted p-value < 0.05) and you need
  to identify hub metabolites or rank pathways by structural network impact rather
  than enrichment p-value alone.
when_to_use_negative:
- When the analysis goal is purely over-representation significance (Fisher's Exact
  Test adjusted p-values) and network topology/topology-based impact is not of interest.
- When the input metabolites cannot be mapped to KEGG compound identifiers or names,
  or when working outside KEGG/STITCH knowledge bases without an alternative pathway-feature
  mapping.
- When the input is already a raw metabolomics matrix without differential analysis
  results — run run_de() first (fc_threshold = 1, pval_threshold = 0.05) to derive
  the significant metabolite query set.
tools:
- name: EnrichMet
  role: Computes relative betweenness centrality for metabolites and topology-based
    pathway impact scores; exposes results as the metabolite_centrality table, rbc_plot,
    and impact_plot
  repo: https://github.com/biodatalab/enrichmet.git
- name: igraph
  role: Constructs the bipartite metabolite-pathway network and the STITCH metabolite-metabolite
    network on which betweenness centrality is computed
- name: STITCH
  role: Source of chemical-chemical relationships used to build the metabolite-metabolite
    interaction network for pathway impact scoring (combined score ≥ 100)
- name: KEGG
  role: Provides the curated pathway-metabolite associations defining the bipartite
    network for metabolite centrality computation
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:a3cd7c49318e8371cec36dcdb3a7c41af314c8ecde2a56417b36d752639abb4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  - STITCH
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# betweenness-centrality-computation

## Summary

Compute betweenness centrality on metabolite-centric networks to identify hub metabolites and quantify pathway impact in EnrichMet. Betweenness is normalized to relative betweenness centrality (RBC) and combined with Fisher ORA significance to prioritize biologically important pathways and metabolites.

## When to use

Use when a metabolomics differential analysis has yielded a significant metabolite list (absolute fold change ≥ 1, adjusted p-value < 0.05) and you need to identify hub metabolites or rank pathways by structural network impact rather than enrichment p-value alone.

## When NOT to use

- When the analysis goal is purely over-representation significance (Fisher's Exact Test adjusted p-values) and network topology/topology-based impact is not of interest.
- When the input metabolites cannot be mapped to KEGG compound identifiers or names, or when working outside KEGG/STITCH knowledge bases without an alternative pathway-feature mapping.
- When the input is already a raw metabolomics matrix without differential analysis results — run run_de() first (fc_threshold = 1, pval_threshold = 0.05) to derive the significant metabolite query set.

## Inputs

- Significant metabolite list (absolute fold change ≥ 1, adjusted p-value < 0.05) as KEGG IDs and/or names
- KEGG pathway-to-metabolite annotation map (PathwayVsMetabolites table, e.g., fetch_kegg_pathway_metabolites(organism = "hsa"))
- STITCH chemical-chemical interaction pairs with combined scores (filtered at combined score ≥ 100) for the pathway impact variant
- Summary statistics table with columns met_id, pval or padj, and log2fc (from run_de() or example_data)

## Outputs

- Metabolite centrality table (S3 data.frame with Display_Name and RBC_Metabolite columns; exported as centrality_table.csv in the case-study workflow)
- Relative Betweenness Centrality (RBC) plot ranking top hub metabolites
- Pathway impact scores per pathway, joined with -log10(adjusted p-values) from Fisher ORA
- Pathway impact plot (significance vs topology-based impact score)

## How to apply

For hub-metabolite detection, build a bipartite metabolite-pathway network in R with igraph from the KEGG pathway-metabolite annotation map (PathwayVsMetabolites), linking each significant metabolite to the KEGG pathways it participates in, then compute betweenness centrality for every metabolite node. Normalize betweenness to relative betweenness centrality (RBC) by dividing each metabolite's betweenness by the maximum betweenness in the network, rank metabolites by RBC, and render the RBC plot (Figure 4A/4B style). For pathway impact, instead build the metabolite-metabolite interaction network from STITCH chemical-chemical relationships filtered at combined score ≥ 100, compute betweenness on that network, and derive the pathway impact score quantifying each pathway's contribution to network connectivity and information flow. Combine impact scores with -log10(adjusted p-values) from the Fisher's Exact Test ORA to render the pathway impact plot (Figure 2B style), highlighting top significantly enriched pathways. The rationale is that ORA significance alone can misprioritize pathways; a pathway with modest adjusted p-value but high structural impact (e.g., caffeine metabolism alongside pyrimidine metabolism in the control vs mutant lung cancer case study) may deserve prioritization.

## Related tools

- **EnrichMet** (Computes relative betweenness centrality for metabolites and topology-based pathway impact scores; exposes results as the metabolite_centrality table, rbc_plot, and impact_plot) — https://github.com/biodatalab/enrichmet.git
- **igraph** (Constructs the bipartite metabolite-pathway network and the STITCH metabolite-metabolite network on which betweenness centrality is computed)
- **STITCH** (Source of chemical-chemical relationships used to build the metabolite-metabolite interaction network for pathway impact scoring (combined score ≥ 100))
- **KEGG** (Provides the curated pathway-metabolite associations defining the bipartite network for metabolite centrality computation)

## Examples

```
results <- enrichmet(inputMetabolites=inputMetabolites, PathwayVsMetabolites=PathwayVsMetabolites, example_data=example_summary, analysis_type=c("enrichment","centrality","network")); head(results$metabolite_centrality[, c("Display_Name","RBC_Metabolite")])
```

## Evaluation signals

- RBC values are bounded in [0, 1] because betweenness is normalized by the maximum betweenness in the network; the top-ranked metabolite should be at or near the maximum.
- In the control vs mutant lung cancer case study, pyruvic acid, arachidonic acid, and L-Tyrosine must rank among the hub metabolites, matching Figures 4A/4B; in the README example, Pyruvate is the top central metabolite (RBC_Metabolite ≈ 0.146).
- For pathway impact, both pyrimidine metabolism (top adjusted p-value) and caffeine metabolism (strong structural impact) must appear among the high-impact significant pathways, matching the Figure 2B pattern.
- Verify the STITCH filter was applied: only chemical-chemical interactions with combined score ≥ 100 enter the network; changing this cutoff should change computed betweenness values.
- The enrichmet() call must return results$metabolite_centrality (with RBC_Metabolite column) and results$rbc_plot / results$impact_plot objects when analysis_type includes 'centrality' and 'network'.

## Limitations

- Results depend on runtime-fetched annotation data (KEGG REST, Reactome mappings, LION via BioPortal) cached with BiocFileCache; database content is not redistributed, so centrality rankings can shift with database versions.
- The STITCH combined score ≥ 100 cutoff directly shapes network topology and hence betweenness/impact scores; different cutoffs yield different hub metabolites.
- Betweenness on the pathway-metabolite bipartite network reflects KEGG pathway membership structure, so hub status is only meaningful for metabolites well-annotated in KEGG.
- No changelog documented for the package.

## Evidence

- [intro] we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [intro] STITCH interactions filtered at combined score >= 100: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [intro] EnrichMet constructs metabolite-metabolite network from STITCH: "we construct a metabolite-metabolite interaction network using the known chemical-chemical relationships from STITCH database"
- [readme] README: computes relative betweenness centrality: "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
- [readme] README: RBC plot lists relative betweenness centrality of query metabolites: "# Relative betweenness centrality of query metabolites
results$rbc_plot"
