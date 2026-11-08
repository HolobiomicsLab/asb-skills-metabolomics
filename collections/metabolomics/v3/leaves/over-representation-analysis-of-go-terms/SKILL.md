---
name: over-representation-analysis-of-go-terms
description: Use when you have a gene list associated with an exposure or network
  module (e.g. chemical-exposome-linked gene nodes from a MetaNet co_net object) and
  need enriched GO biological functions.
when_to_use_negative:
- The goal is network construction or visualization itself (use MetaNet's c_net_calculate/c_net_build/c_net_plot
  workflow) rather than functional enrichment of an already-derived gene list
- The input is a full feature table (e.g. raw transcript abundance matrix) rather
  than a filtered, exposure-associated gene list — ORA needs the associated-gene background
- Gene identifiers cannot be mapped to GO terms, making the enrichment test undefined
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MetaNet
  role: Provides the multi-omics network objects (multi1 / co_net) from which the
    exposure-associated gene lists are extracted before GO over-representation analysis
  repo: github.com/Asa12138/MetaNet
- name: R
  role: Execution environment in which the ReporterScore-style GO over-representation
    analysis is run
- name: ReporterScore
  role: ORA method applied to the exposure-associated gene lists (GO for chemical
    exposome, KEGG for microbial exposome)
- name: KEGG
  role: Curated pathway knowledge base used for the companion KEGG over-representation
    analysis combined with the GO results
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:a20ed40b225ce2cdc4d5fb5d97cf49df14e00fd5b6b441e8c7907ae6cc82bf3c
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# over-representation-analysis-of-go-terms

## Summary

Performs ReporterScore-style Gene Ontology over-representation analysis in R on exposure-associated gene lists extracted from a MetaNet multi-omics network, to identify enriched biological functions (e.g. DNA damage response and cellular stress for the chemical exposome).

## When to use

Use when you have a gene list associated with an exposure or network module (e.g. chemical-exposome-linked gene nodes from a MetaNet co_net object) and need enriched GO biological functions.

## When NOT to use

- The goal is network construction or visualization itself (use MetaNet's c_net_calculate/c_net_build/c_net_plot workflow) rather than functional enrichment of an already-derived gene list
- The input is a full feature table (e.g. raw transcript abundance matrix) rather than a filtered, exposure-associated gene list — ORA needs the associated-gene background
- Gene identifiers cannot be mapped to GO terms, making the enrichment test undefined

## Inputs

- MetaNet multi-omics network object (multi1 / co_net) with gene nodes and exposure associations
- Chemical-exposure-associated gene list (gene symbols/identifiers from network nodes)
- GO ontology annotation for over-representation testing

## Outputs

- GO over-representation results with term identifiers and statistics
- Combined KEGG + GO results table: reporterscore_ora_results.csv

## How to apply

Load the exposome-transcriptome case-study network from the MetaNet package data (multi1 / co_net objects, built with multi_net_build at r_threshold=0.6 over microbiome, metabolome and transcriptome layers) to obtain gene nodes and their exposure associations. Separate the gene nodes into microbial-exposure-associated and chemical-exposure-associated lists so each exposome domain is tested independently. Run ReporterScore-style over-representation analysis in R: GO on the chemical-exposure-associated gene list (and KEGG on the microbial list, per the case study). Extract GO term identifiers and over-representation statistics, expecting disease-related biology such as Parkinson's disease, Alzheimer's disease, DNA damage response and cellular stress for the chemical exposome, and negative immune-regulation terms for the microbial exposome. Write the combined KEGG and GO results to reporterscore_ora_results.csv.

## Related tools

- **MetaNet** (Provides the multi-omics network objects (multi1 / co_net) from which the exposure-associated gene lists are extracted before GO over-representation analysis) — github.com/Asa12138/MetaNet
- **R** (Execution environment in which the ReporterScore-style GO over-representation analysis is run)
- **ReporterScore** (ORA method applied to the exposure-associated gene lists (GO for chemical exposome, KEGG for microbial exposome))
- **KEGG** (Curated pathway knowledge base used for the companion KEGG over-representation analysis combined with the GO results)

## Examples

```
library(MetaNet); data(multi_test); multi1 <- multi_net_build(list(Microbiome=micro, Metabolome=metab, Transcriptome=transc), r_threshold=0.6)
```

## Evaluation signals

- reporterscore_ora_results.csv contains both KEGG and GO over-representation results with term identifiers and statistics
- Chemical-exposome GO results include disease-related biology: Parkinson's disease, Alzheimer's disease, DNA damage response and cellular stress terms
- Microbial-exposome analysis recovers eight significant pathways, most related to negative regulation of immune responses
- Every gene in the tested lists traces back to gene nodes in the multi1 / co_net MetaNet objects
- Microbial and chemical exposome results are kept as separate gene lists/tests, not merged

## Limitations

- MetaNet's c_net_build warns 'Have not do p-value adjustment! use the p.value to build network', so gene lists from thresholded correlations may carry unadjusted-p-value bias into the enrichment input
- Correlation-based network optimization may overlook complex, non-linear, or indirect associations, which constrains how gene-exposure associations are defined before ORA
- Threshold selection for correlation filtering is subjective and arbitrary, potentially biasing the network topology and thus the downstream biological interpretation of enriched terms

## Evidence

- [other] the chemical exposome was linked to disease-related pathways including Parkinson's disease and Alzheimer's disease as well as DNA damage response and cellular stress.: [evidence span withheld — anchor status: not_found]
