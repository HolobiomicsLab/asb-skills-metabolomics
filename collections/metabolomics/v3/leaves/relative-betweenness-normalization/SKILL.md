---
name: relative-betweenness-normalization
description: Use when building a pathway-metabolite interaction network from significant
  metabolites and you need to rank hub metabolites (e.g. pyruvate, pyruvic acid, arachidonic
  acid) by normalized centrality rather than raw betweenness values.
when_to_use_negative:
- The task is raw pathway over-representation significance (Fisher's exact test /
  Enrichment_Ratio) rather than hub identification - centrality normalization is not
  part of that statistic.
- The network topology itself is not built yet and no betweenness values exist; this
  skill assumes a precomputed igraph metabolite-pathway network.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0154
tools:
- name: EnrichMet
  role: Implements the RBC computation as part of its topology-based network analysis
    module, producing a metabolite centrality table and an rbc_plot.
  repo: https://github.com/biodatalab/enrichmet.git
- name: igraph
  role: Used to construct the metabolite-pathway bipartite network and compute betweenness
    centrality on metabolite nodes before normalization.
- name: R
  role: Runtime environment for the igraph-based centrality workflow inside EnrichMet.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2025.08.28.672951
    title: 'EnrichMet: R Package for Integrated Pathway and Network Analysis for Metabolomics'
  source_content_hash: sha256:73dbdadd3b7b7d67257baf038389a607ce68d754f87042d4e910e57eb9188d39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodatalab/enrichmet.git
  tools:
  - EnrichMet
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Relative Betweenness Normalization

## Summary

Normalizes igraph betweenness centrality of metabolite nodes to Relative Betweenness Centrality (RBC) by dividing each metabolite's betweenness by the maximum betweenness in the network, so hub metabolites in a KEGG pathway-metabolite network can be ranked on a comparable 0-1 scale.

## When to use

Use when building a pathway-metabolite interaction network from significant metabolites and you need to rank hub metabolites (e.g. pyruvate, pyruvic acid, arachidonic acid) by normalized centrality rather than raw betweenness values.

## When NOT to use

- The task is raw pathway over-representation significance (Fisher's exact test / Enrichment_Ratio) rather than hub identification - centrality normalization is not part of that statistic.
- The network topology itself is not built yet and no betweenness values exist; this skill assumes a precomputed igraph metabolite-pathway network.

## Inputs

- igraph bipartite network of significant metabolites linked to KEGG pathways (metabolites with absolute fold change >= 1 and adjusted p value < 0.05)
- betweenness centrality values computed for metabolite nodes

## Outputs

- RBC_Metabolite column of normalized betweenness scores (metabolite centrality table)
- ranked list of hub metabolites (e.g. pyruvic acid, arachidonic acid, L-Tyrosine)
- Relative Betweenness Centrality (RBC) plot

## How to apply

Construct a bipartite network linking each significant metabolite (absolute fold change >= 1 and adjusted p value < 0.05) to the KEGG pathways it participates in, using igraph in R. Compute betweenness centrality for every metabolite node on that network, quantifying each metabolite's contribution to overall network connectivity and information flow. Then compute RBC by dividing each metabolite's betweenness by the maximum betweenness observed in the network, yielding a normalized 0-1 score. Rank metabolites by RBC (the EnrichMet output table exposes an 'RBC_Metabolite' column, e.g. Pyruvate at 0.146 in the README example) and render the top hub metabolites as an RBC plot. The rationale is that raw betweenness values are network-size dependent, so dividing by the maximum makes centrality comparable across datasets and pathways.

## Related tools

- **EnrichMet** (Implements the RBC computation as part of its topology-based network analysis module, producing a metabolite centrality table and an rbc_plot.) — https://github.com/biodatalab/enrichmet.git
- **igraph** (Used to construct the metabolite-pathway bipartite network and compute betweenness centrality on metabolite nodes before normalization.)
- **R** (Runtime environment for the igraph-based centrality workflow inside EnrichMet.)

## Examples

```
rbc <- betweenness(g) ; PathwayVsMetabolites$RBC_Metabolite <- rbc / max(rbc)
```

## Evaluation signals

- RBC values are bounded: each metabolite's normalized betweenness lies in [0, 1], with the top-ranked hub equal to 1.
- The metabolite centrality table contains a named 'RBC_Metabolite' column alongside the metabolite identifier (e.g. Display_Name).
- Known hub metabolites such as Pyruvate (RBC 0.146248573 in the README example) or pyruvic acid, arachidonic acid and L-Tyrosine (Figures 4A/4B) appear at the top of the ranking.
- Significant metabolites fed into the network satisfy absolute fold change >= 1 and adjusted p value < 0.05.
- The ranking is scale-invariant: multiplying all betweenness values by a constant leaves the RBC ranking unchanged.

## Limitations

- RBC is defined relative to the maximum betweenness within a single network, so scores are not directly comparable between networks of different sizes or composition.
- The network is built only from KEGG pathway membership (optionally STITCH interactions filtered at combined score >= 100), so hubs reflect that topology and may miss biologically important metabolites absent from KEGG.

## Evidence

- [results] Task-level evidence from source card: [evidence span withheld — anchor status: not_found]
- [intro] Article centrality method: "we calculated the pathway impact score which quantifies the extent of which a pathway contributes to overall network connectivity and information flow"
- [readme] README RBC example output: "computes relative betweenness centrality for metabolites, and performs Metabolite Set Enrichment Analysis (MetSEA) via **fgsea**."
- [intro] Significant metabolite threshold: "We selected significant metabolites (absolute fold change ≥ 1 and adjusted p value < 0.05)"
