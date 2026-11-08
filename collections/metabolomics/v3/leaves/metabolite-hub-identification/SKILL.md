---
name: metabolite-hub-identification
description: Use when a metabolomics differential analysis has produced significant
  metabolites (|fold change| ≥ 1, adjusted p < 0.05) and you need to rank them as
  network hubs by relative betweenness centrality on a KEGG pathway–metabolite network.
when_to_use_negative:
- No differential statistics exist, so there is no principled way to select significant
  metabolites (|FC| ≥ 1, adj p < 0.05) as query nodes for the centrality calculation
- The organism's metabolite–pathway associations are absent from KEGG and no user-supplied
  Pathway/Metabolites table is available (EnrichMet fetches annotations at runtime
  from KEGG REST and does not ship database dumps)
- The goal is pathway-level enrichment significance only (Fisher's exact test / MetSEA),
  not topological hub ranking of individual metabolites
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: EnrichMet
  role: R package that computes relative betweenness centrality for metabolites and
    returns the metabolite_centrality table and RBC plot in a single enrichmet() call
  repo: https://github.com/biodatalab/enrichmet.git
- name: igraph
  role: Used to construct the bipartite pathway–metabolite network and compute betweenness
    centrality for each metabolite node
- name: R
  role: Runtime environment in which EnrichMet and igraph execute the centrality workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:e258b0c17eb61ae532f3b9b578307652dc23b98b3c05fc58408b8f8e05453070
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolite-hub-identification

## Summary

Identifies hub metabolites by computing relative betweenness centrality (RBC) on a bipartite pathway–metabolite network built from KEGG annotations and the significant metabolites from a differential analysis, as demonstrated in the EnrichMet lung cancer case study.

## When to use

Use when a metabolomics differential analysis has produced significant metabolites (|fold change| ≥ 1, adjusted p < 0.05) and you need to rank them as network hubs by relative betweenness centrality on a KEGG pathway–metabolite network.

## When NOT to use

- No differential statistics exist, so there is no principled way to select significant metabolites (|FC| ≥ 1, adj p < 0.05) as query nodes for the centrality calculation
- The organism's metabolite–pathway associations are absent from KEGG and no user-supplied Pathway/Metabolites table is available (EnrichMet fetches annotations at runtime from KEGG REST and does not ship database dumps)
- The goal is pathway-level enrichment significance only (Fisher's exact test / MetSEA), not topological hub ranking of individual metabolites

## Inputs

- Differential analysis summary statistics with metabolite identifiers, pval/padj, and log2 fold change (e.g. summary_stat.csv)
- KEGG pathway-to-metabolite annotation map (Pathway/Metabolites table, e.g. KEGG hsa)
- Query set of significant metabolites as KEGG IDs and/or names (input_type = "kegg", "name", or "mixed")

## Outputs

- Metabolite centrality table ranking metabolites by relative betweenness centrality (RBC_Metabolite), e.g. centrality_table.csv
- Relative Betweenness Centrality (RBC) plot of top hub metabolites
- Pathway–metabolite network graph showing hub metabolites in context

## How to apply

First select significant metabolites using absolute fold change ≥ 1 and adjusted p-value < 0.05 from the differential summary statistics. Link each significant metabolite to the KEGG pathways it participates in using the pathway-to-metabolite annotation map (e.g. fetch_kegg_pathway_metabolites(organism = "hsa")) to form a bipartite network. Compute betweenness centrality for every metabolite node with igraph, then normalize to relative betweenness centrality (RBC) by dividing each metabolite's betweenness by the maximum betweenness in the network. Rank metabolites by RBC, write the centrality table (e.g. centrality_table.csv), and render the top hubs as a Relative Betweenness Centrality plot. In EnrichMet this is done in one call via analysis_type = "centrality", returning the metabolite_centrality data.frame with Display_Name and RBC_Metabolite columns; in the paper's case study pyruvic acid, arachidonic acid, and L-Tyrosine emerged as hubs (Figures 4A/4B).

## Related tools

- **EnrichMet** (R package that computes relative betweenness centrality for metabolites and returns the metabolite_centrality table and RBC plot in a single enrichmet() call) — https://github.com/biodatalab/enrichmet.git
- **igraph** (Used to construct the bipartite pathway–metabolite network and compute betweenness centrality for each metabolite node)
- **R** (Runtime environment in which EnrichMet and igraph execute the centrality workflow)

## Examples

```
results <- enrichmet(inputMetabolites = inputMetabolites, PathwayVsMetabolites = PathwayVsMetabolites, example_data = example_summary, analysis_type = c("centrality")); head(results$metabolite_centrality[, c("Display_Name", "RBC_Metabolite")])
```

## Evaluation signals

- The metabolite_centrality table contains numeric RBC_Metabolite values in [0, 1], since RBC is betweenness divided by the maximum betweenness in the network
- The top-ranked hub by RBC is a biologically central metabolite — in the README example Pyruvate has RBC ≈ 0.146, roughly an order of magnitude above D-Glucose (≈ 0.012)
- Validation against the paper: pyruvic acid, arachidonic acid, and L-Tyrosine rank among the hub metabolites, matching Figures 4A/4B of the control vs mutant lung cancer case study
- Every query metabolite appearing in the centrality table maps to at least one pathway in the KEGG PathwayVsMetabolites map; unmapped metabolites should be excluded or reported
- The RBC plot renders without error and the exported centrality_table.csv row count equals the number of query metabolites placed on the network

## Limitations

- Centrality is computed only for the query (significant) metabolites on the pathway–metabolite network, so hub rankings depend on which metabolites passed the differential filters (|FC| ≥ 1, adj p < 0.05)
- Pathway and compound annotations are fetched at runtime from public APIs (KEGG REST, Reactome mappings) and cached with BiocFileCache; the package does not redistribute KEGG dumps, so results depend on network access and database state
- No changelog is documented for the package, so version-dependent changes to centrality behavior are hard to track

## Evidence

- [intro] Topology-based network analysis computing betweenness centrality: [evidence span withheld — anchor status: not_found]
- [intro] Significant metabolite filters: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
- [readme] EnrichMet computes RBC for metabolites: "computes relative betweenness centrality for metabolites, and performs
Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**"
- [readme] Centrality output columns and example ranking: "results$metabolite_centrality[
    , c("Display_Name", "RBC_Metabolite")
  ]"
- [readme] Annotations fetched at runtime, not shipped: "Pathway and compound annotations are obtained at **runtime** from public
APIs (KEGG REST and Reactome mappings) and cached locally with
**BiocFileCache**"
