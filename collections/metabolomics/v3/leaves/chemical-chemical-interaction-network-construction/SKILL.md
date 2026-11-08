---
name: chemical-chemical-interaction-network-construction
description: Use when a set of significant metabolites (e.g. fold change ≥1, adjusted
  p < 0.05) needs to be interpreted structurally as an interaction network, to identify
  hub metabolites and compare topology against a published network figure.
when_to_use_negative:
- The goal is pathway over-representation or MetSEA rather than metabolite-metabolite
  interaction topology — EnrichMet's Fisher's Exact Test and fgsea modules are the
  appropriate modules instead
- The metabolite set cannot be mapped to STITCH compound identifiers (e.g. lipid species
  without chemical-chemical interaction records; for lipidomics EnrichMet instead
  uses LION lipid-ontology associations)
- Input is raw metabolomics intensity data that has not yet undergone differential
  analysis to define the significant metabolite query set
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: STITCH
  role: Source database of known chemical-chemical interaction records from which
    the metabolite-metabolite interaction network is built
- name: EnrichMet
  role: R package whose topology-based network analysis module builds the interaction
    network and computes betweenness centrality to rank hub metabolites
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Execution environment in which the edge list is built and centrality metrics
    are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:4f20835b32615cff503c0efae3a1b719e5ad33a5a16fb5623a49fe386acfa0f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  - STITCH
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# chemical-chemical-interaction-network-construction

## Summary

Builds a metabolite-metabolite interaction network from STITCH chemical-chemical relationships (filtered at combined score ≥100) and applies EnrichMet topology-based network analysis in R to rank hub metabolites by node degree and betweenness centrality, as demonstrated in the control-vs-mutant lung cancer case study where Xanthine emerged as a hub.

## When to use

Use when a set of significant metabolites (e.g. fold change ≥1, adjusted p < 0.05) needs to be interpreted structurally as an interaction network, to identify hub metabolites and compare topology against a published network figure.

## When NOT to use

- The goal is pathway over-representation or MetSEA rather than metabolite-metabolite interaction topology — EnrichMet's Fisher's Exact Test and fgsea modules are the appropriate modules instead
- The metabolite set cannot be mapped to STITCH compound identifiers (e.g. lipid species without chemical-chemical interaction records; for lipidomics EnrichMet instead uses LION lipid-ontology associations)
- Input is raw metabolomics intensity data that has not yet undergone differential analysis to define the significant metabolite query set

## Inputs

- Significant metabolite list from differential analysis (absolute fold change ≥1 and adjusted p-value <0.05)
- Metabolite names/HMDB or compound identifiers for mapping to STITCH compound IDs
- STITCH chemical-chemical interaction records for the mapped compounds
- Combined-score threshold parameter (≥100)

## Outputs

- Filtered edge list of metabolite-metabolite interactions (combined score ≥100)
- Interaction network graph object (igraph-style edge list)
- Per-metabolite node degree and centrality metrics (e.g. Relative Betweenness Centrality)
- Ranked hub-metabolite list (e.g. Xanthine as top hub)

## How to apply

First map the significant metabolites from the differential analysis (control vs mutant lung cancer dataset, selected at absolute fold change ≥1 and adjusted p-value <0.05) to STITCH compound identifiers. Retrieve the known chemical-chemical interaction records for these compounds from STITCH and filter the interaction pairs at combined score ≥100, the paper's biological-relevance threshold. Build the filtered metabolite-metabolite interaction network as a graph (edge list) in R using EnrichMet's topology-based network analysis module, which computes betweenness centrality (reported as Relative Betweenness Centrality, RBC). Compute node degree and centrality metrics and rank metabolites to identify hubs — in the case study, Xanthine emerged as a hub metabolite. Finally compare the reproduced network topology and hub ranking against the reference network figure (Figure 4C of the paper).

## Related tools

- **STITCH** (Source database of known chemical-chemical interaction records from which the metabolite-metabolite interaction network is built)
- **EnrichMet** (R package whose topology-based network analysis module builds the interaction network and computes betweenness centrality to rank hub metabolites) — https://github.com/biodatalab/enrichmet.git
- **R** (Execution environment in which the edge list is built and centrality metrics are computed)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, reactome_df = reactome_df, analysis_type = c("centrality", "interaction"), input_type = "kegg")
```

## Evaluation signals

- The edge list contains only interaction pairs with STITCH combined score ≥100 — any pair below the threshold indicates a filtering failure
- Every edge endpoint maps back to a significant metabolite from the differential analysis (absolute fold change ≥1, adjusted p <0.05)
- Centrality output reproduces the Relative Betweenness Centrality ranking (RBC_Metabolite column), with the expected hub (Xanthine in the case study) at the top
- Network topology and hub ranking visually match the reference network figure (Figure 4C of the paper)
- Network build should complete in seconds consistent with EnrichMet's benchmark (~3 s for a 94-metabolite dataset), not minutes as with FELLA (~14 min)

## Limitations

- The network depends on STITCH's known chemical-chemical relationships, so metabolites lacking STITCH compound mappings or interaction records are excluded from the topology
- The ≥100 combined-score cutoff is a stated biological-relevance threshold from the paper; other thresholds will change the hub ranking
- No changelog is documented for EnrichMet, so version drift in network module behavior is hard to track

## Evidence

- [intro] STITCH interaction filtering threshold: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [intro] Network construction from STITCH: "we construct a metabolite-metabolite interaction network using the known chemical-chemical relationships from STITCH database"
- [intro] Topology-based analysis rationale: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [readme] EnrichMet computes relative betweenness centrality: "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
- [readme] Interaction network plot output: "**Reactome interaction network plot**"
