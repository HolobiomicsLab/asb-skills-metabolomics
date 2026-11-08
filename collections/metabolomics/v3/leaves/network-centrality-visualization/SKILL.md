---
name: network-centrality-visualization
description: Use when a differential metabolomics hit list (e.g., absolute fold change
  >= 1 and adjusted p < 0.05) needs hub-metabolite prioritization via betweenness
  centrality on a KEGG pathway network.
when_to_use_negative:
- The metabolite list lacks KEGG compound identifiers or mappable names and no kegg_lookup
  is available, so nodes cannot be attached to the KEGG pathway network
- The goal is pathway-level enrichment significance (Fisher's exact test or MetSEA)
  rather than ranking metabolites by network position
- Input is a feature table needing differential analysis and filtering first and no
  significance thresholds have been applied — run run_de() with fc_threshold = 1,
  pval_threshold = 0.05 before centrality
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0092
tools:
- name: EnrichMet
  role: R package whose centrality module computes relative betweenness centrality
    and generates the RBC and network plots
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Runtime environment for the enrichmet() single-call workflow
- name: igraph
  role: Graph library underlying the bipartite pathway-metabolite network on which
    betweenness centrality is computed
- name: KEGG
  role: Source of the curated pathway-metabolite associations that define the network
    nodes and edges
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:0367dc8ffcb813b59b0ea1bec5081cfb56b1404fb04f35c428a48e426dd81bb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# network-centrality-visualization

## Summary

Computes relative betweenness centrality (RBC) for metabolites in a KEGG pathway-metabolite bipartite network to identify hub metabolites, and renders them as the Figure 4A/4B-style centrality plot. This is EnrichMet's topology-based network module for spotting metabolites that drive network connectivity and information flow.

## When to use

Use when a differential metabolomics hit list (e.g., absolute fold change >= 1 and adjusted p < 0.05) needs hub-metabolite prioritization via betweenness centrality on a KEGG pathway network.

## When NOT to use

- The metabolite list lacks KEGG compound identifiers or mappable names and no kegg_lookup is available, so nodes cannot be attached to the KEGG pathway network
- The goal is pathway-level enrichment significance (Fisher's exact test or MetSEA) rather than ranking metabolites by network position
- Input is a feature table needing differential analysis and filtering first and no significance thresholds have been applied — run run_de() with fc_threshold = 1, pval_threshold = 0.05 before centrality

## Inputs

- Character vector of significant metabolites as KEGG compound IDs/names (input_type = 'kegg', 'name', or 'mixed'), or da_results output from run_de()
- Pathway-vs-metabolites annotation table (PathwayVsMetabolites) fetched from KEGG REST at runtime and cached with BiocFileCache
- Optional summary statistics data.frame with columns met_id, pval/padj, log2fc for background correction

## Outputs

- metabolite_centrality table (S3 data.frame) with Display_Name and RBC_Metabolite columns
- Relative Betweenness Centrality (RBC) plot (results$rbc_plot) highlighting top hub metabolites
- Pathway-metabolite network graph (results$network_plot)

## How to apply

Load the KEGG pathway-to-metabolite annotation map (via fetch_kegg_pathway_metabolites, e.g. organism 'hsa') and build a bipartite network in R with igraph linking each significant metabolite to the KEGG pathways it participates in. First filter metabolites at absolute fold change >= 1 and adjusted p value < 0.05 (or pass da_results from run_de(), which applies fc_threshold = 1 and pval_threshold = 0.05). Compute betweenness centrality for every metabolite node, then normalize by dividing each value by the maximum betweenness in the network to obtain relative betweenness centrality (RBC). Rank metabolites by RBC, return the metabolite_centrality table (columns Display_Name, RBC_Metabolite), and render the RBC plot (results$rbc_plot) showing the top hub metabolites. The rationale is that betweenness quantifies each metabolite's contribution to overall network connectivity and information flow, so high-RBC metabolites are pathway-independent hubs worth biological follow-up.

## Related tools

- **EnrichMet** (R package whose centrality module computes relative betweenness centrality and generates the RBC and network plots) — https://github.com/biodatalab/enrichmet.git
- **R** (Runtime environment for the enrichmet() single-call workflow)
- **igraph** (Graph library underlying the bipartite pathway-metabolite network on which betweenness centrality is computed)
- **KEGG** (Source of the curated pathway-metabolite associations that define the network nodes and edges)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, analysis_type = c("centrality", "network")); head(results$metabolite_centrality[, c("Display_Name", "RBC_Metabolite")])
```

## Evaluation signals

- RBC values fall in [0, 1] by construction because each betweenness is divided by the maximum betweenness in the network
- The metabolite_centrality data.frame contains Display_Name and RBC_Metabolite columns and the top row has the largest RBC (e.g., Pyruvate at 0.146248573 in the README example)
- results$rbc_plot and results$network_plot exist as ggplot/ggraph-class plot objects among the eight generated plots
- In the control vs mutant lung cancer case study, pyruvic acid, arachidonic acid, and L-Tyrosine rank among the top hub metabolites, matching Figures 4A/4B
- Every metabolite scored by RBC is connected to at least one KEGG pathway node in the bipartite network (no isolated metabolite nodes)

## Limitations

- Hub ranking depends on the KEGG annotation coverage; metabolites absent from the pathway map contribute no betweenness
- RBC is relative to the maximum betweenness of the specific network, so values are not comparable across different query sets or organisms
- The article documents no changelog for the package, so behavior may drift across versions
- Annotation data are fetched at runtime from KEGG REST and cached with BiocFileCache, so results depend on network access and cache state

## Evidence

- [intro] betweenness quantifies contribution to network connectivity: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [readme] enrichmet computes relative betweenness centrality: "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
- [readme] RBC plot is a dedicated output: "# Relative betweenness centrality of query metabolites
results$rbc_plot"
- [intro] differential filtering thresholds for the metabolite query set: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
