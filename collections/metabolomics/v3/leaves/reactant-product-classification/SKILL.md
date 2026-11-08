---
name: reactant-product-classification
description: Use when a pathway-metabolite association table (met_path) exists with
  KEGG reaction IDs and you must determine each metabolite's role (reactant vs product)
  in its reaction for mechanistic annotation.
when_to_use_negative:
- Metabolites have not been mapped to KEGG compound/reaction IDs yet (e.g. before
  map_keggid/compoundinfo steps) — there is no reaction equation to parse.
- The analysis relies on non-KEGG pathway resources (MetENP's gene-enzyme and reactant/product
  annotation is KEGG-specific via KEGGREST).
- Only pathway enrichment scores are needed and no gene/reaction/enzyme annotation
  is requested.
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: MetENP
  role: R package providing the enzyme_gene_info and react_substrate functions used
    to fetch genes/enzymes and classify metabolites as reactants or products
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: Bioconductor client MetENP uses to query the KEGG REST API for reactions,
    genes, and enzymes of each pathway
  repo: https://github.com/Bioconductor/KEGGREST
- name: KEGG
  role: Reference database supplying the pathway reaction equations parsed to label
    reactants and products
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:c93df6ca3c7c6fc73a954436c2ae97c27e319ea9f179e9337d546dcb08fe14ac
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGG
  - KEGGREST
  - MetENP
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# reactant-product-classification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Classifies each significantly altered metabolite as a reactant (substrate) or product within its associated KEGG reaction, using MetENP's react_substrate function applied after enzyme/gene retrieval.

## When to use

Use when a pathway-metabolite association table (met_path) exists with KEGG reaction IDs and you must determine each metabolite's role (reactant vs product) in its reaction for mechanistic annotation.

## When NOT to use

- Metabolites have not been mapped to KEGG compound/reaction IDs yet (e.g. before map_keggid/compoundinfo steps) — there is no reaction equation to parse.
- The analysis relies on non-KEGG pathway resources (MetENP's gene-enzyme and reactant/product annotation is KEGG-specific via KEGGREST).
- Only pathway enrichment scores are needed and no gene/reaction/enzyme annotation is requested.

## Inputs

- met_path pathway-metabolite association table from the upstream met_pathways step (e.g. for study ST000915)
- df_metenrichment enrichment dataframe annotated with organism code (sps) and metabolite class (classm)
- KEGG reaction ID / pathway linkage per metabolite
- KEGG reaction equation strings fetched via KEGGREST

## Outputs

- Per-metabolite table with reaction ID, gene ID, EC number, and reactant/product flag
- enzyme_gene_reaction_info.csv annotation file

## How to apply

First run MetENP's enzyme_gene_info(df_metenrichment, sps, classm), which uses KEGGREST to query the KEGG REST API for the reactions, organism-specific gene IDs, and EC-number enzymes attached to each associated pathway. Pass the resulting gene/reaction table to react_substrate(met_gene_info), which parses each KEGG reaction equation (the CPD-prefixed stoichiometric strings) and labels every metabolite as reactant (substrate) or product. Assemble per-metabolite rows carrying the reaction ID, gene ID, EC number and reactant/product flag, and verify the recovered totals against a known reference run — for study ST000915 the expected result is 77 reactions, 109 genes and 46 enzymes associated to the significantly altered metabolites. Write the annotated table to enzyme_gene_reaction_info.csv.

## Related tools

- **MetENP** (R package providing the enzyme_gene_info and react_substrate functions used to fetch genes/enzymes and classify metabolites as reactants or products) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (Bioconductor client MetENP uses to query the KEGG REST API for reactions, genes, and enzymes of each pathway) — https://github.com/Bioconductor/KEGGREST
- **KEGG** (Reference database supplying the pathway reaction equations parsed to label reactants and products)

## Examples

```
source('path/react_substrate.R'); met_sub_info <- react_substrate(met_gene_info)
```

## Evaluation signals

- Totals reconcile with the published ST000915 reference run: 77 reactions, 109 genes, and 46 enzymes associated to the significantly altered metabolites.
- Every metabolite row in the output carries exactly one reactant/product label derived from a parsed KEGG reaction equation.
- Each row contains the full quadruple: reaction ID, organism-specific gene ID, EC number, and reactant/product flag.
- Output file enzyme_gene_reaction_info.csv is produced and its row count matches the number of metabolite-reaction associations in met_path.
- Gene IDs and EC numbers returned are consistent with the KEGG pathway entries for the specified organism code (sps).

## Limitations

- Annotation is restricted to metabolite species with available KEGG pathways; metabolites lacking KEGG reaction assignments cannot be labeled.
- Results depend on the upstream met_pathways association table and on species-specific mapping (korg/sps); incorrect organism codes yield wrong gene IDs.
- No software version number or statistical validation metrics are reported for the gene-enzyme/reaction step in the article.

## Evidence

- [readme] KEGGREST provides the KEGG REST API client used for retrieval: [evidence span withheld — anchor status: not_found]
