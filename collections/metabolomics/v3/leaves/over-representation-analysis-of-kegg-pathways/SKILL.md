---
name: over-representation-analysis-of-kegg-pathways
description: Use when a list of genes significantly associated with an exposure layer
  (e.g., microbial or chemical exposome nodes in a MetaNet multi-omics network) needs
  pathway-level functional interpretation via KEGG or GO over-representation testing.
when_to_use_negative:
- The input is a raw feature abundance table or network edges rather than a gene list
  derived from exposure-associated network nodes — no over-representation target exists
- The goal is network construction or visualization itself (MetaNet's c_net_build
  / c_net_plot workflow) rather than functional interpretation of selected genes
- The reference knowledge base needed is not KEGG or GO (e.g., taxonomic or metabolite-level
  pathway analysis on the non-transcriptome omics layers)
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetaNet
  role: provides the multi-omics network (multi1 / co_net) whose gene nodes and exposure
    associations define the ORA gene lists
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: R
  role: runtime environment in which the ReporterScore-style over-representation analysis
    is executed
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: KEGG
  role: curated pathway database used as the reference set for the over-representation
    analysis
- name: ReporterScore
  role: performs the KEGG and GO over-representation analysis on the exposure-associated
    gene lists
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:2fb0e27b4cfefb5c522a2ae37889a0f53be7a2dc6f36b39722d748c9947545c9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KEGG
  - MetaNet
  - R
  - ReporterScore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# over-representation-analysis-of-kegg-pathways

## Summary

Runs KEGG (and GO) over-representation analysis with ReporterScore on gene nodes drawn from the MetaNet exposome–transcriptome network, testing which functional pathways are enriched among genes associated with the microbial versus the chemical exposome.

## When to use

Use when a list of genes significantly associated with an exposure layer (e.g., microbial or chemical exposome nodes in a MetaNet multi-omics network) needs pathway-level functional interpretation via KEGG or GO over-representation testing.

## When NOT to use

- The input is a raw feature abundance table or network edges rather than a gene list derived from exposure-associated network nodes — no over-representation target exists
- The goal is network construction or visualization itself (MetaNet's c_net_build / c_net_plot workflow) rather than functional interpretation of selected genes
- The reference knowledge base needed is not KEGG or GO (e.g., taxonomic or metabolite-level pathway analysis on the non-transcriptome omics layers)

## Inputs

- MetaNet multi-omics network object (multi1 / co_net) with gene nodes and exposure associations from the case-study package data
- List of genes significantly associated with the microbial exposome
- List of genes significantly associated with the chemical exposome
- KEGG and GO pathway reference databases

## Outputs

- reporterscore_ora_results.csv containing combined KEGG and GO over-representation statistics
- Pathway identifiers and statistics for the eight microbial-exposome immune-regulation pathways
- Pathway identifiers and statistics for chemical-exposome disease-related pathways (Parkinson's, Alzheimer's, DNA damage response)

## How to apply

Load the exposome–transcriptome network from the MetaNet case-study package data (multi1 / co_net objects) and extract the gene nodes together with their exposure associations. Build the gene list for each exposure layer, then run ReporterScore-style KEGG over-representation analysis on the microbial-exposure-associated genes and ReporterScore-style GO over-representation analysis on the chemical-exposure-associated genes in R. Extract pathway identifiers and statistics for the eight immune-regulation pathways (microbial exposome, mostly negative regulation of immune responses) and the disease-related pathways for the chemical exposome (Parkinson's disease, Alzheimer's disease, DNA damage response and cellular stress). Write the combined KEGG and GO results to reporterscore_ora_results.csv.

## Related tools

- **MetaNet** (provides the multi-omics network (multi1 / co_net) whose gene nodes and exposure associations define the ORA gene lists) — https://cran.r-project.org/web/packages/MetaNet
- **R** (runtime environment in which the ReporterScore-style over-representation analysis is executed) — https://cran.r-project.org/web/packages/MetaNet
- **KEGG** (curated pathway database used as the reference set for the over-representation analysis)
- **ReporterScore** (performs the KEGG and GO over-representation analysis on the exposure-associated gene lists)

## Examples

```
data("multi_test", package = "pcutils"); multi1 <- multi_net_build(list(Microbiome = micro, Metabolome = metab, Transcriptome = transc), r_threshold = 0.6)
```

## Evaluation signals

- reporterscore_ora_results.csv exists and contains both KEGG and GO pathway rows with identifiers and statistics
- Exactly eight pathways are reported as significantly associated with the microbial exposome
- The chemical-exposome results include the expected disease-related pathways (Parkinson's disease, Alzheimer's disease, DNA damage response)
- Genes fed to the ORA match the gene nodes of the MetaNet multi1 / co_net network objects with exposure associations
- Microbial-exposome pathways are dominated by negative regulation of immune responses terms, matching the published finding

## Limitations

- Correlation-based network optimization may overlook complex, non-linear, or indirect associations, affecting which genes enter the ORA lists
- Threshold selection for correlation filtering is typically subjective and arbitrary, potentially biasing downstream biological interpretation
- No data availability or accession information for the case-study datasets is provided in the article section, hindering direct reproduction

## Evidence

- [results] eight microbial pathways finding: [evidence span withheld — anchor status: not_found]
- [readme] multi-omics network construction in README: [evidence span withheld — anchor status: external_doc]
