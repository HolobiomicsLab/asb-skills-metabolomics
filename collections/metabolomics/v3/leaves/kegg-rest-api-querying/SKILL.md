---
name: kegg-rest-api-querying
description: Use when a pathway-metabolite association table (met_path) already exists
  and you need gene IDs, EC numbers, reaction IDs, and reactant/product roles for
  the metabolites in those pathways.
when_to_use_negative:
- Metabolites have not yet been mapped to KEGG compound IDs (run map_keggid/compoundinfo
  upstream first)
- The species of choice has no KEGG pathway coverage — MetENP explicitly supports
  only 'all metabolite species with available KEGG pathways'
- You only need pathway enrichment scores, not gene/enzyme/reaction annotation — use
  path_enrichmentscore instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KEGGREST
  role: R/Bioconductor client that issues the KEGG REST API queries for reactions,
    genes, and enzymes underlying enzyme_gene_info
  repo: https://github.com/Bioconductor/KEGGREST
- name: MetENP
  role: Provides the enzyme_gene_info and react_substrate functions that wrap the
    KEGGREST queries and reaction-equation parsing
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: R
  role: Execution environment for the MetENP functions and Bioconductor dependencies
  repo: https://github.com/metabolomicsworkbench/MetENP
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:2de41e255198b0a6e4eba5bd754db0b1c958fc3202bafa0ad98b93cd9fe4e3e5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/Bioconductor/KEGGREST
  tools:
  - KEGG
  - KEGGREST
  - MetENP
  - R
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# kegg-rest-api-querying

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Annotate significantly altered metabolites with their KEGG reactions, organism-specific genes, and EC enzymes by querying the KEGG REST API through KEGGREST, then classify each metabolite as a reactant (substrate) or product in its reaction via react_substrate.

## When to use

Use when a pathway-metabolite association table (met_path) already exists and you need gene IDs, EC numbers, reaction IDs, and reactant/product roles for the metabolites in those pathways.

## When NOT to use

- Metabolites have not yet been mapped to KEGG compound IDs (run map_keggid/compoundinfo upstream first)
- The species of choice has no KEGG pathway coverage — MetENP explicitly supports only 'all metabolite species with available KEGG pathways'
- You only need pathway enrichment scores, not gene/enzyme/reaction annotation — use path_enrichmentscore instead

## Inputs

- Pathway-metabolite association table (met_path / df_metenrichment) from the upstream met_pathways step
- KEGG organism species code (sps) for the species of choice
- Metabolite class specification (classm)

## Outputs

- Annotated gene/reaction/enzyme table (enzyme_gene_reaction_info.csv) with per-metabolite rows of reaction ID, organism-specific gene ID, EC number, and reactant/product flag

## How to apply

Start from the met_path table produced by the upstream met_pathways step for the study of interest (e.g., ST000915). Call MetENP's enzyme_gene_info(df_metenrichment, sps, classm), which uses KEGGREST to query the KEGG REST API for the reactions, organism-specific gene IDs, and EC-number enzymes attached to each associated pathway. Pass the resulting gene/reaction table to react_substrate(met_gene_info), which parses each KEGG reaction equation and labels every metabolite as a reactant (substrate) or product. Assemble per-metabolite rows carrying reaction ID, gene ID, EC number, and the reactant/product flag, and check recovered totals against reported counts (for ST000915: 77 reactions, 109 genes, 46 enzymes). Write the annotated table to enzyme_gene_reaction_info.csv.

## Related tools

- **KEGGREST** (R/Bioconductor client that issues the KEGG REST API queries for reactions, genes, and enzymes underlying enzyme_gene_info) — https://github.com/Bioconductor/KEGGREST
- **MetENP** (Provides the enzyme_gene_info and react_substrate functions that wrap the KEGGREST queries and reaction-equation parsing) — https://github.com/metabolomicsworkbench/MetENP
- **R** (Execution environment for the MetENP functions and Bioconductor dependencies) — https://github.com/metabolomicsworkbench/MetENP

## Examples

```
source('path/enzyme_gene_info.R'); source('path/react_substrate.R'); met_gene_info <- enzyme_gene_info(df_metenrichment, sps, classm); rxn_flagged <- react_substrate(met_gene_info)
```

## Evaluation signals

- Recovered totals match the reported counts for the study (ST000915: 77 reactions, 109 genes, 46 enzymes associated to significantly altered metabolites)
- Every metabolite row carries a reactant or product label parsed from its KEGG reaction equation — no missing flags
- Gene IDs are organism-specific and consistent with the sps code passed to enzyme_gene_info
- EC numbers are present for enzymatic entries and correspond to the queried KEGG reactions
- Output file enzyme_gene_reaction_info.csv is produced with the expected columns (reaction ID, gene ID, EC number, reactant/product flag)

## Limitations

- Requires live connectivity to the KEGG REST API; coverage is limited to metabolite species with available KEGG pathways
- Depends on Bioconductor packages KEGGREST, KEGGgraph, and pathview, which must be installed via BiocManager; KEGGREST maintainers advise not installing directly from GitHub
- No software version number or statistical validation metrics are reported in the article for this step

## Evidence

- [readme] KEGGREST is an R/Bioconductor package that provides a client interface to the Kyoto Encyclopedia of Genes and Genomes (KEGG) REST API.: [evidence span withheld — anchor status: not_found]
