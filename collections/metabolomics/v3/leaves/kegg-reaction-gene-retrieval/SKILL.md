---
name: kegg-reaction-gene-retrieval
description: Use when a metabolite enrichment table (e.g. MetENP met_path output for
  a study like ST000915) has associated KEGG pathways and you need the underlying
  genes, reactions, EC enzymes, and each metabolite's reactant/product role.
when_to_use_negative:
- When no KEGG pathway association exists for the metabolites of interest — the skill
  depends on pathway IDs already recovered by met_pathways.
- When you only need enrichment scores or pathway plots and not gene/enzyme/reaction-level
  functional annotation.
- When metabolites are not mapped to KEGG compound IDs (e.g. only RefMet names without
  map_keggid conversion).
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_0092
tools:
- name: MetENP
  role: Provides the enzyme_gene_info and react_substrate functions that retrieve
    genes/enzymes and label reactant/product roles
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: R/Bioconductor client interface used by enzyme_gene_info to query the KEGG
    REST API for reactions, genes, and enzymes
  repo: https://github.com/Bioconductor/KEGGREST
- name: R
  role: Execution environment for the MetENP workflow functions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:41cd126c25b7d3dc744363ea5f3be43bb9a19d8a3f7c370a496e0f15d3dc7178
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGG
  - KEGGREST
  - MetENP
  - R
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# kegg-reaction-gene-retrieval

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Retrieves genes, reactions, and EC-number enzymes attached to KEGG pathways associated with significantly altered metabolites using MetENP's enzyme_gene_info (via KEGGREST), then labels each metabolite as reactant (substrate) or product in its reaction using react_substrate.

## When to use

Use when a metabolite enrichment table (e.g. MetENP met_path output for a study like ST000915) has associated KEGG pathways and you need the underlying genes, reactions, EC enzymes, and each metabolite's reactant/product role.

## When NOT to use

- When no KEGG pathway association exists for the metabolites of interest — the skill depends on pathway IDs already recovered by met_pathways.
- When you only need enrichment scores or pathway plots and not gene/enzyme/reaction-level functional annotation.
- When metabolites are not mapped to KEGG compound IDs (e.g. only RefMet names without map_keggid conversion).

## Inputs

- Pathway-metabolite association table (met_path) from the upstream met_pathways step
- MetENP enrichment data frame (df_metenrichment) with metabolite classes and enrichment scores
- Organism species code (sps) for KEGG REST queries
- Metabolite class specification (classm)

## Outputs

- Annotated per-metabolite table with reaction ID, gene ID, EC number, and reactant/product flag
- enzyme_gene_reaction_info.csv
- Counts of associated reactions, genes, and enzymes (e.g. 77 reactions, 109 genes, 46 enzymes for ST000915)

## How to apply

Load the pathway-metabolite association table (met_path) produced by the upstream met_pathways step for the target study. Call MetENP's enzyme_gene_info(df_metenrichment, sps, classm), which uses KEGGREST to query the KEGG REST API for the reactions, organism-specific gene IDs, and EC-number enzymes attached to each associated pathway; sps specifies the organism. Pass the resulting gene/reaction table to react_substrate(met_gene_info), which parses each KEGG reaction equation and labels every metabolite as a reactant (substrate) or product. Assemble per-metabolite rows carrying reaction ID, gene ID, EC number, and the reactant/product flag, and cross-check recovered totals against known counts — for study ST000915 this step recovered 77 reactions, 109 genes, and 46 enzymes. Write the annotated table to enzyme_gene_reaction_info.csv.

## Related tools

- **MetENP** (Provides the enzyme_gene_info and react_substrate functions that retrieve genes/enzymes and label reactant/product roles) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (R/Bioconductor client interface used by enzyme_gene_info to query the KEGG REST API for reactions, genes, and enzymes) — https://github.com/Bioconductor/KEGGREST
- **R** (Execution environment for the MetENP workflow functions)

## Examples

```
source('path/enzyme_gene_info.R'); source('path/react_substrate.R'); met_gene_info <- enzyme_gene_info(df_metenrichment, sps, classm); rxn_info <- react_substrate(met_gene_info)
```

## Evaluation signals

- Total reaction, gene, and enzyme counts recovered match reported values (77 reactions, 109 genes, 46 enzymes for ST000915).
- Every metabolite row carries a non-empty reactant/product flag derived from a parsed KEGG reaction equation.
- Enzyme entries are valid EC numbers and gene IDs are organism-specific KEGG gene IDs.
- Output file enzyme_gene_reaction_info.csv contains the columns reaction ID, gene ID, EC number, and reactant/product flag per metabolite.
- Every row traces back to a KEGG pathway ID present in the input met_path table.

## Limitations

- Coverage is limited to metabolite species with available KEGG pathways; metabolites outside KEGG or with missing pathway associations yield no gene/enzyme annotation.
- react_substrate relies on parsing KEGG reaction equations, so ambiguous or malformed equation strings can produce wrong or missing reactant/product labels.
- Retrieval requires network access to the KEGG REST API via KEGGREST.

## Evidence

- [methods] MetENP's enzyme_gene_info function retrieves genes and enzymes involved in the associated KEGG pathways: [evidence span withheld — anchor status: not_found]
