---
name: topology-based-pathway-impact-scoring
description: Use when a differential metabolomics query (e.g., control vs mutant)
  yields statistically enriched KEGG pathways but topology must decide which significant
  pathway is biologically most influential, as in prioritizing caffeine metabolism
  alongside pyrimidine metabolism.
when_to_use_negative:
- When the analysis goal is purely statistical over-representation without any network/topology
  prioritization — plain Fisher's exact test ORA suffices.
- When query metabolites cannot be mapped to KEGG compound identifiers and no interaction
  network source (STITCH or Reactome mappings) is available, since centrality cannot
  be computed.
- When the input is already a pathway-level result table with no metabolite-level
  members, preventing reconstruction of the metabolite-metabolite interaction network.
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: EnrichMet
  role: R package that computes relative betweenness centrality for metabolites, derives
    pathway impact scores, and renders the pathway impact plot within a single function
    call
  repo: https://github.com/biodatalab/enrichmet.git
- name: STITCH
  role: Source of chemical-chemical relationships used to construct the metabolite-metabolite
    interaction network, filtered at combined score >= 100
- name: KEGG
  role: Provides curated pathway-to-metabolite annotations against which impact scores
    and adjusted p-values are joined
- name: R
  role: Execution environment; network construction, centrality computation, and impact
    plotting are all run as R code
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:fb923bba4b1f6fbeb42a8a09dcade9c5ebd00f52647cbc4455652b6b60031540
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - KEGG
  - R
  - STITCH
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# topology-based-pathway-impact-scoring

## Summary

Prioritizes metabolomics pathways beyond statistical enrichment by computing a topology-based pathway impact score from betweenness centrality on a metabolite-metabolite interaction network, then combining impact with -log10(adjusted p-values) from Fisher's exact test ORA.

## When to use

Use when a differential metabolomics query (e.g., control vs mutant) yields statistically enriched KEGG pathways but topology must decide which significant pathway is biologically most influential, as in prioritizing caffeine metabolism alongside pyrimidine metabolism.

## When NOT to use

- When the analysis goal is purely statistical over-representation without any network/topology prioritization — plain Fisher's exact test ORA suffices.
- When query metabolites cannot be mapped to KEGG compound identifiers and no interaction network source (STITCH or Reactome mappings) is available, since centrality cannot be computed.
- When the input is already a pathway-level result table with no metabolite-level members, preventing reconstruction of the metabolite-metabolite interaction network.

## Inputs

- ORA pathway enrichment results (Fisher's exact test adjusted p-values on KEGG pathways) from upstream differential analysis
- Significant metabolite list (KEGG compound IDs/names), typically filtered at absolute fold change >= 1 and adjusted p-value < 0.05
- STITCH chemical-chemical interaction network (combined score >= 100) or Reactome reaction mappings (reactome_df) used by enrichmet to build the interaction graph
- KEGG pathway-to-metabolite map (PathwayVsMetabolites, e.g., organism = 'hsa')

## Outputs

- Pathway impact score per pathway (contribution to network connectivity and information flow)
- Relative betweenness centrality (RBC) per metabolite (results$metabolite_centrality table)
- Pathway impact plot (impact_plot: -log10(adjusted p-value) vs topology-based impact score, Figure 2B style)

## How to apply

Build a metabolite-metabolite interaction network from STITCH chemical-chemical relationships, filtering interactions to combined score >= 100 to ensure biological relevance. Compute betweenness centrality on the network and derive the pathway impact score, which quantifies the extent to which a pathway contributes to overall network connectivity and information flow. Combine impact scores with -log10(adjusted p-values) from the Fisher's exact test ORA (Benjamini-Hochberg adjusted) and render a pathway impact plot (Figure 2B style) highlighting top significantly enriched pathways. In enrichmet, the same analysis is exposed as the 'centrality'/'network' analysis types, which compute relative betweenness centrality (RBC) for query metabolites and produce an impact_plot showing significance vs. topology-based impact. Validate by confirming that the top adjusted-p-value pathway (e.g., pyrimidine metabolism) and a strong-structural-impact pathway (e.g., caffeine metabolism) both appear among high-impact significant pathways.

## Related tools

- **EnrichMet** (R package that computes relative betweenness centrality for metabolites, derives pathway impact scores, and renders the pathway impact plot within a single function call) — https://github.com/biodatalab/enrichmet.git
- **STITCH** (Source of chemical-chemical relationships used to construct the metabolite-metabolite interaction network, filtered at combined score >= 100)
- **KEGG** (Provides curated pathway-to-metabolite annotations against which impact scores and adjusted p-values are joined)
- **R** (Execution environment; network construction, centrality computation, and impact plotting are all run as R code)

## Examples

```
PathwayVsMetabolites <- fetch_kegg_pathway_metabolites(organism="hsa"); results <- enrichmet(inputMetabolites=inputMetabolites, PathwayVsMetabolites=PathwayVsMetabolites, example_data=example_summary, kegg_lookup=kegg_lookup, reactome_df=reactome_df, analysis_type=c("centrality","network")); results$impact_plot
```

## Evaluation signals

- Both the top adjusted-p-value pathway (pyrimidine metabolism in the control vs mutant lung cancer case study) and the strong-structural-impact pathway (caffeine metabolism) appear among high-impact significant pathways, matching the reported Figure 2B pattern.
- All network edges derive from interactions passing the STITCH combined score >= 100 filter; verify the filter was applied before centrality computation.
- results$metabolite_centrality contains a non-empty RBC_Metabolite column with values in [0, 1] (relative betweenness centrality), e.g., pyruvate = 0.146 in the README example.
- The impact plot axes correspond to -log10(adjusted p-value) vs topology-based impact score, with top significantly enriched pathways highlighted.
- Impact scores are defined only for pathways containing query metabolites present in the interaction network; unmapped pathways should not receive spurious impact values.

## Limitations

- Annotation and interaction data are fetched at runtime from public APIs (KEGG REST and Reactome mappings) and cached with BiocFileCache, so the analysis requires network access (or a pre-populated cache) and user-supplied Pathway/Metabolites tables otherwise.
- Impact scoring is anchored to KEGG pathway structure; the paper notes the framework supports extensions to other knowledge bases (e.g., LION for lipidomics), but lipid ontology content is not shipped and must be fetched on demand.
- No changelog is documented for the package, so version drift in runtime-fetched annotations may affect reproducibility.

## Evidence

- [intro] Topology-based network analysis computing betweenness centrality and pathway impact scores: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [intro] STITCH interaction filter: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [readme] enrichmet centrality module: "**enrichmet** performs pathway over-representation analysis using Fisher’s exact test (optionally restricted to a measured background), computes relative betweenness centrality for metabolites"
- [readme] impact plot definition: "# Pathway impact: significance vs. topology-based impact score
results$impact_plot"
- [other] case study finding: [evidence span withheld — anchor status: not_found]
