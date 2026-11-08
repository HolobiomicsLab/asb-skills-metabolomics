---
name: microbial-vs-chemical-exposome-pathway-comparison
description: Use when you have an exposure–transcriptome network (e.g., MetaNet co_net/multi1
  objects) and need to compare KEGG/GO over-representation of genes associated with
  different exposure types (microbial vs chemical).
when_to_use_negative:
- Input is a raw feature table or expression matrix with no built network — build
  the co-occurrence/exposome network first (e.g., c_net_calculate/c_net_build or multi_net_build)
  before gene selection
- Goal is network construction, layout, or topology rather than pathway-level interpretation
  of exposure-associated genes
- Case-study dataset accessions are unavailable — the article provides no data availability
  information for the two case study datasets, so only the bundled multi_test package
  data can substitute
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: MetaNet
  role: Provides the package case-study data (multi_test/multi1/co_net) from which
    exposure-associated gene nodes are extracted for enrichment analysis
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: R
  role: Execution environment for installing MetaNet (devtools) and running the ReporterScore-style
    ORA scripts
- name: ReporterScore
  role: ORA method used to perform KEGG and GO over-representation analysis on the
    exposure-associated gene lists
- name: KEGG
  role: Pathway knowledge base supplying pathway identifiers for the microbial and
    chemical exposome ORA results
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:addd6796dd1d724854d7da6912562c3cf5f83b86f616a0136f5c780f04dcc9f2
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# microbial-vs-chemical-exposome-pathway-comparison

## Summary

Reproduces the MetaNet exposome–transcriptome case study in which genes linked to the microbial versus chemical exposome are subjected to ReporterScore-style KEGG and GO over-representation analysis, contrasting immune-regulation pathways with disease and DNA-damage pathways.

## When to use

Use when you have an exposure–transcriptome network (e.g., MetaNet co_net/multi1 objects) and need to compare KEGG/GO over-representation of genes associated with different exposure types (microbial vs chemical).

## When NOT to use

- Input is a raw feature table or expression matrix with no built network — build the co-occurrence/exposome network first (e.g., c_net_calculate/c_net_build or multi_net_build) before gene selection
- Goal is network construction, layout, or topology rather than pathway-level interpretation of exposure-associated genes
- Case-study dataset accessions are unavailable — the article provides no data availability information for the two case study datasets, so only the bundled multi_test package data can substitute

## Inputs

- MetaNet package case-study data objects (multi_test -> multi1 / co_net) containing the exposome–transcriptome network with >40,000 features
- List of genes significantly associated with the microbial exposome
- List of genes significantly associated with the chemical exposome
- KEGG pathway database for over-representation analysis
- GO term database for over-representation analysis

## Outputs

- reporterscore_ora_results.csv with combined KEGG and GO over-representation statistics
- Pathway identifiers and statistics for the eight immune-regulation pathways (microbial exposome)
- Pathway identifiers and statistics for Parkinson's, Alzheimer's, and DNA-damage-response pathways (chemical exposome)

## How to apply

Load the MetaNet case-study package data (multi1 / co_net objects) to obtain gene nodes and their exposure associations. Run ReporterScore-style KEGG over-representation analysis in R on the microbial-exposure-associated gene list, and ReporterScore-style GO over-representation analysis on the chemical-exposure-associated gene list. Extract pathway identifiers and statistics for the eight immune-regulation pathways (microbial exposome, mostly negative regulation of immune responses) and the disease-related pathways for the chemical exposome (Parkinson's disease, Alzheimer's disease, DNA damage response, cellular stress). Write the combined KEGG and GO results to reporterscore_ora_results.csv. Judge correctness by verifying that the eight immune-related KEGG terms and the disease/DNA-damage terms appear with significant over-representation statistics in the output table.

## Related tools

- **MetaNet** (Provides the package case-study data (multi_test/multi1/co_net) from which exposure-associated gene nodes are extracted for enrichment analysis) — https://cran.r-project.org/web/packages/MetaNet
- **R** (Execution environment for installing MetaNet (devtools) and running the ReporterScore-style ORA scripts)
- **ReporterScore** (ORA method used to perform KEGG and GO over-representation analysis on the exposure-associated gene lists)
- **KEGG** (Pathway knowledge base supplying pathway identifiers for the microbial and chemical exposome ORA results)

## Examples

```
devtools::install_github("Asa12138/MetaNet"); data("multi_test"); multi1 <- multi_net_build(list(Microbiome=micro, Metabolome=metab, Transcriptome=transc))
```

## Evaluation signals

- reporterscore_ora_results.csv exists and contains both KEGG and GO result blocks
- Exactly eight pathways are extracted for the microbial exposome, all related to negative regulation of immune responses
- Chemical-exposome results include Parkinson's disease, Alzheimer's disease, DNA damage response, and cellular stress terms
- Each reported pathway row has a resolvable KEGG or GO identifier plus over-representation statistics
- Gene lists derive from the multi1/co_net network nodes, not from the simulated multi_test demo data alone

## Limitations

- The article provides no data availability or accession information for the two case study datasets, limiting exact reproduction to bundled package data
- The README's multi_test data are a simulated multi-omics dataset ('We create a simulated multi-omics dataset for demonstration purposes'), so enrichment results may not match the published figure exactly
- Correlation-based network construction 'may overlook complex, non-linear, or indirect associations', and threshold selection is typically subjective, which propagates into the gene lists fed to ORA

## Evidence

- [readme] README shows how to load the multi-omics case data and build the network object (multi1) that supplies the gene nodes.: [evidence span withheld — anchor status: external_doc]
- [full_text] Source card workflow defines the ORA procedure, extraction of the eight immune-regulation and disease pathways, and the CSV output.: [evidence span withheld — anchor status: not_found]
