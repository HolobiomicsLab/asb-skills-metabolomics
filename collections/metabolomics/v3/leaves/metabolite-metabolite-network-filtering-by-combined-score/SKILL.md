---
name: metabolite-metabolite-network-filtering-by-combined-score
description: Use when a metabolomics study reports significant metabolites (e.g. absolute
  fold change ≥ 1, adjusted p < 0.05) and you need to rank hubs or compute centrality
  on a chemical-chemical interaction network rather than just pathway enrichment.
when_to_use_negative:
- The analysis goal is plain pathway over-representation or MetSEA without any network/centrality
  layer — filtering STITCH interactions is unnecessary overhead there.
- Your metabolite list contains no STITCH-mappable compound identifiers, so the chemical-chemical
  interaction retrieval cannot proceed.
- You are working with lipidomics via the LION ontology extension, where EnrichMet
  uses lipid-ontology associations instead of STITCH chemical-chemical interactions.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3314
tools:
- name: EnrichMet
  role: Provides the topology-based network analysis module that ingests the filtered
    STITCH interaction graph, computes relative betweenness centrality for metabolites,
    and returns the centrality table and RBC plot used to identify hubs.
  repo: https://github.com/biodatalab/enrichmet.git
- name: STITCH
  role: Source database of known chemical-chemical relationships; its interaction
    pairs are retrieved and filtered at combined score ≥ 100 to form the metabolite-metabolite
    network.
- name: R
  role: Runtime environment in which the enrichmet package is installed (BiocManager::install)
    and the network graph and centrality computation are executed.
  repo: https://github.com/biodatalab/enrichmet.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:c00548b3f98920b3cd20d49361c3b35fd8270db6e41f8a8fc7dc3dbfbe475da3
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
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolite-metabolite-network-filtering-by-combined-score

## Summary

Build a metabolite-metabolite interaction network from STITCH chemical-chemical relationships and filter edges by combined score ≥ 100 so that only biologically relevant compound interactions enter EnrichMet's topology-based network analysis, enabling hub identification (e.g. Xanthine in the lung cancer case study).

## When to use

Use when a metabolomics study reports significant metabolites (e.g. absolute fold change ≥ 1, adjusted p < 0.05) and you need to rank hubs or compute centrality on a chemical-chemical interaction network rather than just pathway enrichment.

## When NOT to use

- The analysis goal is plain pathway over-representation or MetSEA without any network/centrality layer — filtering STITCH interactions is unnecessary overhead there.
- Your metabolite list contains no STITCH-mappable compound identifiers, so the chemical-chemical interaction retrieval cannot proceed.
- You are working with lipidomics via the LION ontology extension, where EnrichMet uses lipid-ontology associations instead of STITCH chemical-chemical interactions.

## Inputs

- List of significant metabolites from differential analysis (fold change ≥ 1, adjusted p < 0.05), as KEGG compound IDs or names
- STITCH chemical-chemical interaction records for those compounds
- R environment with the enrichmet package installed via BiocManager

## Outputs

- Filtered metabolite-metabolite interaction network (edge list / graph object)
- Metabolite centrality table (metabolite_centrality data.frame with RBC values)
- Relative Betweenness Centrality (RBC) plot
- Ranked hub metabolites (e.g. Xanthine identified as a hub in the lung cancer case study)

## How to apply

First select significant metabolites from the differential analysis (absolute fold change ≥ 1 and adjusted p value < 0.05) and map them to STITCH compound identifiers. Retrieve known chemical-chemical interaction records from the STITCH database for those compounds, then filter the interaction pairs at combined score ≥ 100 — the paper's threshold 'to ensure biological relevance'. Construct the metabolite-metabolite network as a graph (edge list) in R via the EnrichMet topology-based network analysis module, and compute relative betweenness centrality (RBC) for metabolites to rank hubs. The EnrichMet function returns a metabolite centrality table (e.g. columns Display_Name, RBC_Metabolite) plus an rbc_plot visualisation; results are compared against the paper's figure of the case-study network to confirm hub metabolites such as Xanthine.

## Related tools

- **EnrichMet** (Provides the topology-based network analysis module that ingests the filtered STITCH interaction graph, computes relative betweenness centrality for metabolites, and returns the centrality table and RBC plot used to identify hubs.) — https://github.com/biodatalab/enrichmet.git
- **STITCH** (Source database of known chemical-chemical relationships; its interaction pairs are retrieved and filtered at combined score ≥ 100 to form the metabolite-metabolite network.)
- **R** (Runtime environment in which the enrichmet package is installed (BiocManager::install) and the network graph and centrality computation are executed.) — https://github.com/biodatalab/enrichmet.git

## Examples

```
Rscript -e 'BiocManager::install("enrichmet"); library(enrichmet); results <- enrichmet(inputMetabolites=inputMetabolites, PathwayVsMetabolites=PathwayVsMetabolites, example_data=example_summary, kegg_lookup=kegg_lookup, reactome_df=reactome_df, analysis_type=c("centrality","network"))'
```

## Evaluation signals

- Every retained edge in the network has STITCH combined score ≥ 100 — any lower-score pair indicates the filter was skipped.
- Input metabolites satisfy the differential-analysis cutoffs (absolute fold change ≥ 1 and adjusted p value < 0.05) before mapping to STITCH IDs.
- enrichmet() output contains a metabolite_centrality data.frame with Display_Name and RBC_Metabolite columns, and an rbc_plot object is generated.
- Hub ranking is reproducible: re-running the enrichment on the same significant-metabolite set yields the same top-ranked metabolites (e.g. Xanthine as a hub in the lung cancer case study).
- The reproduced network topology and hub ranking match the paper's figure of the control-vs-mutant lung cancer case study.

## Limitations

- STITCH interactions are filtered at combined score ≥ 100; scores below this cutoff are discarded, so weaker chemical-chemical relationships are invisible to the network.
- Only metabolites that can be mapped to STITCH compound identifiers enter the network — unmapped significant metabolites are excluded from centrality computation.
- The tool's benchmarking and case study (94-metabolite lung cancer dataset, ~3 s runtime) were on a small human metabolite set; behaviour on much larger or non-human compound lists is not characterised in the article.
- No changelog was documented in the article, so version-specific behaviour of the network module cannot be tracked from the paper alone.

## Evidence

- [intro] we construct a metabolite-metabolite interaction network using the known chemical-chemical relationships from STITCH database: "we construct a metabolite-metabolite interaction network using the known chemical-chemical relationships from STITCH database"
- [intro] To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100).: "To ensure biological relevance, STITCH interactions were filtered based on combined score (≥100)."
- [intro] We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05): "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [readme] **enrichmet** performs pathway over-representation analysis using Fisher's exact test (optionally restricted to a measured background), computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**.: "enrichmet performs pathway over-representation analysis using Fisher's exact test (optionally restricted to a measured background), computes relative betweenness centrality for metabolites"
- [intro] This study used a dataset generated from metabolomic profiles comparing control and mutant lung cancer: "This study used a dataset generated from metabolomic profiles comparing control and mutant lung cancer"
