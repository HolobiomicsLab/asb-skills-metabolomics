---
name: stitch-database-retrieval
description: Use when you have a list of significant metabolites (e.g. control vs
  mutant lung cancer metabolites with |fold change| ≥ 1 and adjusted p < 0.05) and
  need chemical-chemical interaction edges to build a network and identify hub metabolites
  like Xanthine.
when_to_use_negative:
- You only have pathway annotations (KEGG) and no chemical-chemical interaction data
  — use plain pathway over-representation instead
- Input is a metabolomics peak table that has not yet been reduced to a significant-metabolite
  list — differential filtering must happen first
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3174
tools:
- name: STITCH
  role: Source of known chemical-chemical interaction records used to build the metabolite-metabolite
    interaction network
- name: EnrichMet
  role: R package whose topology-based network analysis module builds the filtered
    edge list into a graph and computes node degree and centrality to identify hubs
  repo: https://github.com/biodatalab/enrichmet.git
- name: R
  role: Language in which EnrichMet network analysis is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:7ff5da09c147a37a9c98137d3c2763c7f1327f50a18e7e184c19e7b8d419a40a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  - STITCH
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# stitch-database-retrieval

## Summary

Retrieve and filter known chemical-chemical interaction records from the STITCH database for a set of significant metabolites, then build a metabolite-metabolite interaction graph for topology-based hub analysis in EnrichMet.

## When to use

Use when you have a list of significant metabolites (e.g. control vs mutant lung cancer metabolites with |fold change| ≥ 1 and adjusted p < 0.05) and need chemical-chemical interaction edges to build a network and identify hub metabolites like Xanthine.

## When NOT to use

- You only have pathway annotations (KEGG) and no chemical-chemical interaction data — use plain pathway over-representation instead
- Input is a metabolomics peak table that has not yet been reduced to a significant-metabolite list — differential filtering must happen first

## Inputs

- List of significant metabolites from differential analysis (e.g. |fold change| ≥ 1 and adjusted p-value < 0.05), mapped to STITCH compound identifiers
- STITCH chemical-chemical interaction records for those metabolites

## Outputs

- Edge list of metabolite-metabolite interactions filtered at combined score ≥ 100
- Network graph (R graph object) with node degree and betweenness centrality metrics
- Ranked list of hub metabolites (e.g. Xanthine)

## How to apply

Map each significant metabolite to its STITCH compound identifier, then retrieve the known chemical-chemical interaction records for those metabolites from the STITCH database. Filter the interaction pairs to those with combined score ≥ 100, the paper's stated threshold for biological relevance, discarding weaker edges. Build the filtered pair list into an edge-list graph in R using the EnrichMet topology-based network analysis module. Compute node degree and betweenness centrality to rank metabolites; a high-ranking node such as Xanthine indicates a hub. Compare the resulting network topology and hub ranking against the expected reference (e.g. Figure 4C of the paper).

## Related tools

- **STITCH** (Source of known chemical-chemical interaction records used to build the metabolite-metabolite interaction network)
- **EnrichMet** (R package whose topology-based network analysis module builds the filtered edge list into a graph and computes node degree and centrality to identify hubs) — https://github.com/biodatalab/enrichmet.git
- **R** (Language in which EnrichMet network analysis is executed)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, analysis_type = c("network"), network_top_n = 8)
```

## Evaluation signals

- Every retained edge has STITCH combined score ≥ 100 — count edges below the cutoff and confirm zero
- Graph node set equals the mapped significant metabolites; no unmapped or dangling compounds remain
- Betweenness centrality ranking surfaces Xanthine (or the expected hub) among the top nodes, matching the reference network (Figure 4C)
- Edge count drops substantially after the combined-score filter compared with the unfiltered retrieval

## Limitations

- STITCH interaction retrieval is limited to compounds with known chemical-chemical relationships; metabolites without STITCH entries drop out of the network
- The combined score ≥ 100 cutoff is a stated biological-relevance threshold, not a statistical test — changing it changes the hub ranking
- Network topology (degree, betweenness) depends on the quality and completeness of STITCH's curated chemical-chemical relationships

## Evidence

- [methods] STITCH interactions filtered by combined score >= 100: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [methods] Construction of metabolite-metabolite interaction network from STITCH chemical-chemical relationships: "we construct a metabolite-metabolite interaction network using the known chemical-chemical relationships from STITCH database"
- [methods] Significant metabolites filtered at absolute fold change >= 1 and adjusted p-value < 0.05: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [methods] Topology-based network analysis computing betweenness centrality and pathway impact scores: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [readme] Relative betweenness centrality output in EnrichMet: "computes relative betweenness centrality for metabolites"
