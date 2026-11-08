---
name: network-topology-hub-identification
description: Use when significant metabolites from a case-control metabolomics comparison
  must be placed in a chemical-chemical interaction network to identify hub (high-degree/central)
  metabolites.
when_to_use_negative:
- The goal is pathway over-representation or MetSEA on KEGG pathways rather than chemical-chemical
  interaction topology — use EnrichMet's Fisher's exact test or fgsea modules instead
- Interaction data comes from Reactome reaction mappings rather than STITCH chemical-chemical
  relationships, since the combined score >= 100 filter is STITCH-specific
- Metabolites cannot be mapped to STITCH compound identifiers, so the interaction
  query would return an empty or unrepresentative network
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: EnrichMet
  role: Provides the topology-based network analysis module used to build the metabolite-metabolite
    interaction network and compute betweenness centrality metrics in R
  repo: https://github.com/biodatalab/enrichmet.git
- name: STITCH
  role: Source of the known chemical-chemical interaction records from which the metabolite-metabolite
    network is constructed, filtered at combined score >= 100
- name: R
  role: Execution environment for building the edge-list graph and ranking metabolites
    by centrality
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:99f9bfa06e1f19d1186bd18d6579c932ec1dadbd242f7e7c80b2811a6edf1e98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  - STITCH
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# network-topology-hub-identification

## Summary

Builds a metabolite-metabolite interaction network from STITCH chemical-chemical relationships (combined score >= 100) and computes node degree/betweenness centrality to rank hub metabolites, as demonstrated by identifying Xanthine as a hub in the lung cancer control-vs-mutant case study.

## When to use

Use when significant metabolites from a case-control metabolomics comparison must be placed in a chemical-chemical interaction network to identify hub (high-degree/central) metabolites.

## When NOT to use

- The goal is pathway over-representation or MetSEA on KEGG pathways rather than chemical-chemical interaction topology — use EnrichMet's Fisher's exact test or fgsea modules instead
- Interaction data comes from Reactome reaction mappings rather than STITCH chemical-chemical relationships, since the combined score >= 100 filter is STITCH-specific
- Metabolites cannot be mapped to STITCH compound identifiers, so the interaction query would return an empty or unrepresentative network

## Inputs

- Significant metabolite list from differential analysis (absolute fold change >= 1, adjusted p-value < 0.05)
- STITCH chemical-chemical interaction records for the mapped metabolites
- Metabolite-to-STITCH compound ID mapping
- Edge list for the filtered interaction network

## Outputs

- Metabolite-metabolite interaction network (graph/edge list) filtered at STITCH combined score >= 100
- Per-metabolite centrality table including relative betweenness centrality and node degree
- Ranked hub-metabolite list (e.g. Xanthine as top hub)
- Network topology comparison against Figure 4C of the paper

## How to apply

Map the significant metabolites (selected at absolute fold change >= 1 and adjusted p-value < 0.05) to STITCH compound identifiers, then retrieve known chemical-chemical interaction records for those metabolites from the STITCH database. Filter interaction pairs at combined score >= 100, the paper's stated threshold 'to ensure biological relevance', and build the metabolite-metabolite interaction network as an edge-list graph using EnrichMet's topology-based network analysis module in R. Compute node-level topology metrics — degree and betweenness centrality (EnrichMet reports relative betweenness centrality) — and rank metabolites to identify hubs. Confirm the resulting hub ranking (e.g. Xanthine emerging as a hub in the lung cancer dataset) against the paper's network figure (Figure 4C).

## Related tools

- **EnrichMet** (Provides the topology-based network analysis module used to build the metabolite-metabolite interaction network and compute betweenness centrality metrics in R) — https://github.com/biodatalab/enrichmet.git
- **STITCH** (Source of the known chemical-chemical interaction records from which the metabolite-metabolite network is constructed, filtered at combined score >= 100)
- **R** (Execution environment for building the edge-list graph and ranking metabolites by centrality)

## Examples

```
results_da <- run_de(metabolomics_mat, "TK-CMV", "K-CMV", fc_threshold = 1, pval_threshold = 0.05); results <- enrichmet(inputMetabolites = NULL, PathwayVsMetabolites = PathwayVsMetabolites, da_results = results_da, reactome_df = reactome_df)
```

## Evaluation signals

- Xanthine emerges as a hub metabolite in the chemical-chemical interaction network for the lung cancer control-vs-mutant dataset, matching Figure 4C of the paper
- All retained interaction pairs have STITCH combined score >= 100 (no edges below the biological-relevance threshold)
- Network topology (edge count, node degrees, centrality ranking) reproduces the paper's network figure
- Node degree and betweenness centrality are computable for every network node, with relative betweenness centrality values reported per metabolite
- Input metabolites correspond to the significant set (absolute fold change >= 1, adjusted p-value < 0.05)

## Limitations

- Hub identification depends on STITCH coverage of the mapped metabolites; unmapped compounds are absent from the network
- The combined score >= 100 filter reflects the paper's stated biological-relevance criterion; other thresholds change network density and hub ranking
- No changelog was documented for EnrichMet
- Centrality results depend on which significant metabolites are used as the query set; different fold-change or p-value cutoffs yield different networks

## Evidence

- [intro] STITCH filter threshold: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [intro] Network construction from STITCH relationships: "we construct a metabolite-metabolite interaction network using the known chemical-chemical relationships from STITCH database"
- [readme] Topology metric computed by the package: "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
- [readme] Differential analysis thresholds feeding the network: "run_de(
  metabolomics_mat,
  "TK-CMV",
  "K-CMV",
  fc_threshold = 1,
  pval_threshold = 0.05
)"
- [intro] Significant metabolite selection criteria: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
