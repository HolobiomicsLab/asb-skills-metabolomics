---
name: exposome-transcriptome-gene-set-association
description: Use when a built multi-omics network (e.g., MetaNet multi1 or co_net
  objects) contains gene nodes linked to microbial or chemical exposures and you need
  KEGG/GO over-representation analysis per exposure class.
when_to_use_negative:
- The input is a raw feature abundance table (e.g., otutab or multi_test matrices)
  that has not yet been converted into a built network with exposure-associated gene
  vertices identified.
- The goal is network construction, layout, or visualization of the multi-omics graph
  rather than functional interpretation of exposure-associated gene sets.
- The gene list is the entire transcriptome rather than the network-derived subset
  of exposure-associated genes, which would invalidate the over-representation background.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: MetaNet
  role: Provides the built integrative exposome–transcriptome network (multi1 / co_net
    case-study objects) whose gene vertices and exposure associations define the ORA
    gene lists.
  repo: https://cran.r-project.org/web/packages/MetaNet
- name: R
  role: Execution environment for loading the MetaNet network objects and running
    the ReporterScore-style KEGG/GO over-representation analyses.
- name: KEGG
  role: Curated pathway knowledge base used as the term set for the microbial-exposure
    over-representation analysis.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.06.26.661636
    title: 'MetaNet: a scalable and integrated tool for reproducible omics network
      analysis'
  source_content_hash: sha256:a373de45586651697459855eaba6da5384db6ba190246a953dafa4a75d573b41
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - KEGG
  - MetaNet
  - R
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# exposome-transcriptome-gene-set-association

## Summary

Extracts gene nodes with significant exposure associations from an integrative exposome–transcriptome network built with MetaNet, then runs ReporterScore-style KEGG and GO over-representation analysis separately on the microbial- and chemical-exposure-associated gene lists to identify enriched pathways.

## When to use

Use when a built multi-omics network (e.g., MetaNet multi1 or co_net objects) contains gene nodes linked to microbial or chemical exposures and you need KEGG/GO over-representation analysis per exposure class.

## When NOT to use

- The input is a raw feature abundance table (e.g., otutab or multi_test matrices) that has not yet been converted into a built network with exposure-associated gene vertices identified.
- The goal is network construction, layout, or visualization of the multi-omics graph rather than functional interpretation of exposure-associated gene sets.
- The gene list is the entire transcriptome rather than the network-derived subset of exposure-associated genes, which would invalidate the over-representation background.

## Inputs

- MetaNet multi-omics network object (multi1 / co_net) with gene vertices and their exposure associations
- Microbial-exposure-associated gene list (gene identifiers/symbols from network vertices)
- Chemical-exposure-associated gene list (gene identifiers/symbols from network vertices)
- KEGG pathway database (for microbial-exposure ORA) and GO term ontology (for chemical-exposure ORA)

## Outputs

- reporterscore_ora_results.csv with combined KEGG and GO over-representation results
- Pathway identifiers and statistics for the eight microbial-exposure immune-regulation pathways
- Pathway identifiers and statistics for chemical-exposure disease-related pathways (Parkinson's disease, Alzheimer's disease, DNA damage response, cellular stress)

## How to apply

Load the MetaNet case-study network data (multi_test / multi1 or co_net objects) and subset the network's gene vertices, keeping only genes significantly associated with the microbial exposome or the chemical exposome. Run ReporterScore-style KEGG over-representation analysis on the microbial-exposure gene list and ReporterScore-style GO over-representation analysis on the chemical-exposure gene list, both in R. Extract pathway identifiers and enrichment statistics, keeping distinct pathway sets per exposure class (e.g., immune-regulation pathways vs. disease pathways), and write the combined KEGG and GO results to reporterscore_ora_results.csv. The rationale is that the exposome–transcriptome network's cross-layer edges define which transcripts respond to biological versus chemical exposures, so pathway enrichment is performed on network-derived gene sets rather than a whole-transcriptome background.

## Related tools

- **MetaNet** (Provides the built integrative exposome–transcriptome network (multi1 / co_net case-study objects) whose gene vertices and exposure associations define the ORA gene lists.) — https://cran.r-project.org/web/packages/MetaNet
- **R** (Execution environment for loading the MetaNet network objects and running the ReporterScore-style KEGG/GO over-representation analyses.)
- **KEGG** (Curated pathway knowledge base used as the term set for the microbial-exposure over-representation analysis.)

## Examples

```
data("multi_test"); multi1 <- multi_net_build(list(Microbiome=micro, Metabolome=metab, Transcriptome=transc), r_threshold=0.6)
```

## Evaluation signals

- The microbial-exposure ORA returns eight significantly enriched pathways, most of which relate to negative regulation of immune responses.
- The chemical-exposure ORA returns disease-related pathways including Parkinson's disease and Alzheimer's disease plus DNA damage response and cellular stress terms.
- reporterscore_ora_results.csv exists and contains both KEGG and GO result rows with pathway identifiers and enrichment statistics.
- Enriched gene sets contain only gene vertices present in the source MetaNet network, not the full transcriptome feature set.
- All samples and features of the multi-omics network matched during construction (MetaNet reports 'All samples matched.' / 'All features are OK.') before gene lists were derived.

## Limitations

- Correlation-based network construction may overlook complex, non-linear, or indirect associations, so exposure–gene links in the network may miss some biologically relevant gene sets.
- Threshold selection for correlation filtering is subjective and can bias network topology and downstream biological interpretation, including which genes enter the enrichment analysis.
- MetaNet network building warns 'Have not do p-value adjustment! use the p.value to build network.', so unadjusted p-values determine edge (and hence gene-list) membership.

## Evidence

- [readme] README multi-omics network build that produces the multi1 object whose gene vertices feed the enrichment analysis.: [evidence span withheld — anchor status: not_found]
