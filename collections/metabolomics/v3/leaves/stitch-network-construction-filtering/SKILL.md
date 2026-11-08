---
name: stitch-network-construction-filtering
description: Use when metabolite-level pathway analysis needs network topology (betweenness
  centrality / pathway impact) built on STITCH chemical-chemical interactions, e.g.
  to prioritize pathways beyond ORA p-values.
when_to_use_negative:
- When the analysis goal is over-representation or MetSEA only, with no topology-based
  pathway impact prioritization needed
- When the metabolites cannot be mapped to KEGG compound IDs or the STITCH database
  has no chemical-chemical coverage for the compound classes of interest
- When the analysis concerns gene/protein (not chemical-chemical) interaction networks
  — STITCH chemical-chemical edges are specific to metabolites
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: EnrichMet
  role: R package that consumes the filtered STITCH network to compute betweenness
    centrality, pathway impact scores, and the pathway impact plot
  repo: https://github.com/biodatalab/enrichmet.git
- name: STITCH
  role: Source of known chemical-chemical relationships used to construct the metabolite-metabolite
    interaction network
- name: KEGG
  role: Curated pathway/compound annotations used to assign metabolites to pathways
    when computing impact scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:ccb178b459dece1fb242e32ac03a40091064f5b337f046bd154603ca231eb83f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  - STITCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# stitch-network-construction-filtering

## Summary

Builds the metabolite-metabolite interaction network from STITCH chemical-chemical relationships filtered at combined score >= 100, as used by EnrichMet to derive betweenness centrality and topology-based pathway impact scores for metabolomics pathway prioritization.

## When to use

Use when metabolite-level pathway analysis needs network topology (betweenness centrality / pathway impact) built on STITCH chemical-chemical interactions, e.g. to prioritize pathways beyond ORA p-values.

## When NOT to use

- When the analysis goal is over-representation or MetSEA only, with no topology-based pathway impact prioritization needed
- When the metabolites cannot be mapped to KEGG compound IDs or the STITCH database has no chemical-chemical coverage for the compound classes of interest
- When the analysis concerns gene/protein (not chemical-chemical) interaction networks — STITCH chemical-chemical edges are specific to metabolites

## Inputs

- Query metabolite list with KEGG compound IDs (e.g. C00031, C00022) from differential analysis of the metabolomics matrix
- STITCH chemical-chemical interaction relationships with combined_score attribute
- ORA pathway enrichment results (Fisher's Exact Test adjusted p-values on KEGG pathways) from the upstream enrichment step

## Outputs

- Filtered metabolite-metabolite interaction network (STITCH edges with combined score >= 100)
- Per-metabolite betweenness centrality values (relative betweenness centrality)
- Pathway impact scores per KEGG pathway
- Pathway impact plot (significance vs topology-based impact score, Figure 2B style)

## How to apply

Map query metabolites to KEGG compound IDs, then retrieve known chemical-chemical relationships from the STITCH database and filter them at combined score >= 100 to retain biologically relevant edges. Construct the metabolite-metabolite interaction network from the filtered edges, then compute betweenness centrality on the network and derive the pathway impact score, which quantifies how much each pathway contributes to overall network connectivity and information flow. Combine the impact scores with -log10(adjusted p-values) from the Fisher's Exact Test ORA results (Benjamini-Hochberg adjusted) to render the pathway impact plot highlighting top significantly enriched pathways. Rationale: statistical enrichment alone (e.g. pyrimidine metabolism ranking top by adjusted p-value) can miss pathways with strong structural impact, such as caffeine metabolism in the control vs mutant lung cancer case study, so network topology provides a complementary prioritization signal.

## Related tools

- **EnrichMet** (R package that consumes the filtered STITCH network to compute betweenness centrality, pathway impact scores, and the pathway impact plot) — https://github.com/biodatalab/enrichmet.git
- **STITCH** (Source of known chemical-chemical relationships used to construct the metabolite-metabolite interaction network)
- **KEGG** (Curated pathway/compound annotations used to assign metabolites to pathways when computing impact scores)

## Evaluation signals

- Every retained edge in the interaction network has combined_score >= 100; edges below the cutoff are excluded
- Network nodes correspond to the mapped KEGG compound IDs of the query/measured metabolites
- Betweenness centrality and pathway impact scores are computed for all pathways represented in the network, and impact scores combine with -log10(adjusted p-values) in the impact plot
- In the lung cancer control vs mutant case study, both pyrimidine metabolism (top adjusted p-value) and caffeine metabolism (strong structural impact) appear among high-impact significant pathways, matching the Figure 2B pattern
- The filtered network retains connectivity sufficient to compute informative betweenness centrality (e.g. central metabolites like pyruvate show the highest relative betweenness centrality)

## Limitations

- The combined score >= 100 cutoff discards lower-confidence STITCH interactions, which may omit biologically relevant but weakly supported edges
- Network coverage depends entirely on STITCH chemical-chemical relationships; metabolites absent from STITCH contribute no topology information
- No changelog is documented for the upstream package, and pathway/compound annotations are fetched at runtime from public APIs and cached locally, so results can vary if the databases change or network access fails

## Evidence

- [intro] STITCH interactions were filtered based on combined score (≥100).: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [intro] we construct a metabolite-metabolite interaction network using the known chemical-chemical relationships from STITCH database: "we construct a metabolite-metabolite interaction network using the known chemical-chemical relationships from STITCH database"
- [intro] we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [readme] computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA): "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
