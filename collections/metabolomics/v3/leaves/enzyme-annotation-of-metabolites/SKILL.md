---
name: enzyme-annotation-of-metabolites
description: Use when a pathway-metabolite association table (met_path) for significantly
  altered metabolites must be extended with gene IDs, EC numbers, and reaction roles,
  e.g. to infer enzymatic activity of altered metabolites in a species-specific KEGG
  context.
when_to_use_negative:
- When the species of interest has no KEGG pathways available — MetENP only includes
  support for metabolite species with available KEGG pathways
- Before the upstream met_pathways step has produced the pathway-metabolite association
  table for significantly altered metabolites
- When only metabolite-class or pathway enrichment scores are needed without gene/enzyme-level
  annotation
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: MetENP
  role: Provides the enzyme_gene_info and react_substrate functions that retrieve
    KEGG genes/enzymes and parse reaction equations to label metabolites as reactants
    or products
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: R/Bioconductor client through which enzyme_gene_info queries the KEGG REST
    API for reactions, genes, and EC numbers of associated pathways
  repo: https://bioconductor.org/packages/KEGGREST
- name: KEGG
  role: Reference database supplying the reaction equations, organism-specific gene
    IDs, and EC enzyme numbers attached to each associated pathway
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:d657ce15ead5e0976215cd36fd83a89630f9b55aeea4f1a06965c82cd2d9dfa8
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# enzyme-annotation-of-metabolites

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Annotates significantly altered metabolites with their associated KEGG reactions, organism-specific genes, and EC enzyme numbers, then labels each metabolite as a reactant (substrate) or product in its associated reaction using MetENP's enzyme_gene_info and react_substrate functions.

## When to use

Use when a pathway-metabolite association table (met_path) for significantly altered metabolites must be extended with gene IDs, EC numbers, and reaction roles, e.g. to infer enzymatic activity of altered metabolites in a species-specific KEGG context.

## When NOT to use

- When the species of interest has no KEGG pathways available — MetENP only includes support for metabolite species with available KEGG pathways
- Before the upstream met_pathways step has produced the pathway-metabolite association table for significantly altered metabolites
- When only metabolite-class or pathway enrichment scores are needed without gene/enzyme-level annotation

## Inputs

- Pathway-metabolite association table (met_path / df_metenrichment) from the upstream met_pathways step
- KEGG organism code (sps) for the species of choice
- Metabolite class definition (classm) used to group enriched metabolites

## Outputs

- Per-metabolite annotation table with reaction ID, organism-specific gene ID, EC number, and reactant/product flag (e.g. enzyme_gene_reaction_info.csv)
- Counts of reactions, genes, and enzymes associated with the significantly altered metabolites

## How to apply

Start from the met_path table produced by the upstream met_pathways step, which holds the pathway-metabolite associations for significantly altered metabolites. Call MetENP's enzyme_gene_info(df_metenrichment, sps, classm), which uses KEGGREST to query the KEGG REST API for the reactions, organism-specific gene IDs, and EC-number enzymes attached to each associated pathway. Pass the resulting gene/reaction table to react_substrate(met_gene_info), which parses each KEGG reaction equation and labels every metabolite as reactant (substrate) or product. Assemble per-metabolite rows carrying reaction ID, gene ID, EC number, and the reactant/product flag, and write the annotated table (e.g. enzyme_gene_reaction_info.csv). For study ST000915 this step recovered 77 reactions, 109 genes, and 46 enzymes for the significantly altered metabolites, providing the expected scale check. The species argument (sps) must match a KEGG organism code since gene IDs are organism-specific.

## Related tools

- **MetENP** (Provides the enzyme_gene_info and react_substrate functions that retrieve KEGG genes/enzymes and parse reaction equations to label metabolites as reactants or products) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (R/Bioconductor client through which enzyme_gene_info queries the KEGG REST API for reactions, genes, and EC numbers of associated pathways) — https://bioconductor.org/packages/KEGGREST
- **KEGG** (Reference database supplying the reaction equations, organism-specific gene IDs, and EC enzyme numbers attached to each associated pathway)

## Examples

```
source('path/enzyme_gene_info.R'); source('path/react_substrate.R'); met_gene_info <- enzyme_gene_info(df_metenrichment, sps, classm); react_substrate(met_gene_info)
```

## Evaluation signals

- Recovered totals match the reported benchmark for ST000915: 77 reactions, 109 genes, and 46 enzymes associated to significantly altered metabolites
- Every metabolite row carries the four expected fields: reaction ID, gene ID, EC number, and a reactant/product flag
- Gene IDs are organism-specific KEGG gene identifiers consistent with the sps argument passed to enzyme_gene_info
- All annotated reactions belong to pathways present in the upstream met_path association table for the significant metabolites
- reactant/product labels are consistent with parsing of the corresponding KEGG reaction equations

## Limitations

- Coverage depends entirely on KEGG: only metabolites with KEGG compound IDs and species with available KEGG pathways can be annotated
- Annotation requires live queries to the KEGG REST API via KEGGREST, so results depend on the current state of KEGG entries
- No software version, license statement, or statistical validation metrics are reported for the MetENP workflow in the article

## Evidence

- [abstract] extending the metabolomics enrichment analysis to include species-specific pathway analysis, pathway enrichment scores, gene-enzyme information: "extending the metabolomics enrichment analysis to include species-specific pathway analysis, pathway enrichment scores, gene-enzyme information"
- [discussion] h) obtain enzymatic activities on the metabolites: "h) obtain enzymatic activities on the metabolites"
- [methods] source('path/enzyme_gene_info.R'): "source('path/enzyme_gene_info.R')"
- [readme] MetENP package depends on following Bioconductor packages to function properly: KEGGREST, KEGGgraph, and pathview.: "MetENP package depends on following Bioconductor packages to function properly: KEGGREST, KEGGgraph, and pathview."
