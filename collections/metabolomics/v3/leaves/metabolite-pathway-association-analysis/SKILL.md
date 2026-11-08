---
name: metabolite-pathway-association-analysis
description: Use when you have a metabolite-pathway association table (e.g., MetENP's
  met_path output from a Metabolomics Workbench study such as ST000915) and need to
  recover the reactions, organism-specific gene IDs, and EC numbers behind each pathway,
  plus whether each metabolite is a reactant or product.
when_to_use_negative:
- The metabolites have no available KEGG pathway entries or no KEGG compound IDs (MetENP
  support is limited to 'all metabolite species with available KEGG pathways').
- You only need metabolite-class or pathway enrichment scores (use metclassenrichment
  or path_enrichmentscore instead) and do not need gene/enzyme/reaction-level annotation.
- Your organism is not in MetENP's korg list of KEGG-supported species, so organism-specific
  gene IDs cannot be resolved.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3172
tools:
- name: MetENP
  role: R package providing enzyme_gene_info and react_substrate functions for gene/enzyme/reaction
    annotation of pathway-associated metabolites
  repo: https://github.com/metabolomicsworkbench/MetENP
- name: KEGGREST
  role: Bioconductor client used by enzyme_gene_info to query the KEGG REST API for
    reactions, genes, and enzymes per pathway
  repo: https://github.com/Bioconductor/KEGGREST
- name: KEGG
  role: Reference database supplying pathway, reaction equation, gene, and EC-number
    annotations
- name: Metabolomics Workbench
  role: Source of the study data (e.g., ST000915) whose significantly altered metabolites
    feed the met_path/enrichment tables
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2020.11.20.391912
    title: 'MetENP/MetENPWeb: An R package and web application for metabolomics enrichment
      and pathway analysis in Metabolomics Workbench'
  source_content_hash: sha256:085a2bb1dca090105a6ec3a73a867aa7b670b1a8ad82e6ffaa7db97b9593176d
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/metabolomicsworkbench/MetENP
  tools:
  - KEGG
  - KEGGREST
  - MetENP
  - Metabolomics Workbench
  - R
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45
n_claims: 20
---

# metabolite-pathway-association-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

MetENP workflow step that links significantly altered metabolites to KEGG pathways for a chosen species and then annotates each associated pathway/reaction with genes, EC enzymes, and reactant/product roles via KEGGREST queries to the KEGG REST API.

## When to use

Use when you have a metabolite-pathway association table (e.g., MetENP's met_path output from a Metabolomics Workbench study such as ST000915) and need to recover the reactions, organism-specific gene IDs, and EC numbers behind each pathway, plus whether each metabolite is a reactant or product.

## When NOT to use

- The metabolites have no available KEGG pathway entries or no KEGG compound IDs (MetENP support is limited to 'all metabolite species with available KEGG pathways').
- You only need metabolite-class or pathway enrichment scores (use metclassenrichment or path_enrichmentscore instead) and do not need gene/enzyme/reaction-level annotation.
- Your organism is not in MetENP's korg list of KEGG-supported species, so organism-specific gene IDs cannot be resolved.

## Inputs

- Pathway-metabolite association table (met_path) from MetENP's met_pathways step for a study such as ST000915
- Metabolite enrichment data frame (df_metenrichment) with significantly altered metabolites and metabolite class
- KEGG organism code (sps) for the species of choice, e.g., human
- KEGG reaction equation records retrieved via KEGGREST

## Outputs

- Per-metabolite annotation table with reaction ID, organism-specific gene ID, EC number (enzyme), and reactant/product flag (e.g., enzyme_gene_reaction_info.csv)
- Totals of associated reactions, genes, and enzymes (e.g., 77 reactions, 109 genes, 46 enzymes for ST000915)

## How to apply

Load the pathway-metabolite association table (met_path) produced by the upstream MetENP met_pathways step, then call enzyme_gene_info(df_metenrichment, sps, classm), which uses KEGGREST to query the KEGG REST API for reactions, organism-specific gene IDs, and EC-number enzymes attached to each associated pathway. Pass the resulting gene/reaction table to react_substrate(met_gene_info), which parses each KEGG reaction equation and labels every metabolite as reactant (substrate) or product. Assemble per-metabolite rows carrying reaction ID, gene ID, EC number, and the reactant/product flag. The organism (sps) must be a KEGG organism code supported by MetENP's korg lookup, since gene IDs are species-specific. Rationale: this extends enrichment analysis from pathways to mechanistic functional annotation, one of MetENP's stated capabilities ('gene-enzyme information, and enzymatic activities of the significantly altered' metabolites).

## Related tools

- **MetENP** (R package providing enzyme_gene_info and react_substrate functions for gene/enzyme/reaction annotation of pathway-associated metabolites) — https://github.com/metabolomicsworkbench/MetENP
- **KEGGREST** (Bioconductor client used by enzyme_gene_info to query the KEGG REST API for reactions, genes, and enzymes per pathway) — https://github.com/Bioconductor/KEGGREST
- **KEGG** (Reference database supplying pathway, reaction equation, gene, and EC-number annotations)
- **Metabolomics Workbench** (Source of the study data (e.g., ST000915) whose significantly altered metabolites feed the met_path/enrichment tables)

## Examples

```
source('path/enzyme_gene_info.R'); source('path/react_substrate.R'); met_gene_info <- enzyme_gene_info(df_metenrichment, sps, classm); rxn_info <- react_substrate(met_gene_info)
```

## Evaluation signals

- Recovered totals match the reported counts for the study (77 reactions, 109 genes, 46 enzymes for ST000915).
- Every metabolite row carries a non-empty reaction ID, gene ID, EC number, and a reactant/product flag.
- reactant/product labels agree with parsed KEGG reaction equations: the metabolite appears on the substrate side (reactant) or product side (product) of the equation.
- Gene IDs are organism-specific and consistent with the sps KEGG organism code passed to enzyme_gene_info.
- Output table serializes cleanly as enzyme_gene_reaction_info.csv with one row per metabolite-reaction-gene-enzyme combination.

## Limitations

- Depends on KEGG coverage: metabolites and pathways without KEGG entries yield no gene/enzyme/reaction annotation.
- KEGGREST requires network access to the KEGG REST API; queries are per-pathway and can be slow for many associated pathways (the README loads tictoc for timing).
- No statistical validation metrics or quantitative benchmarks are reported in the paper for the gene/enzyme annotation step; correctness is checked by reconciling counts against the reported totals.

## Evidence

- [results] MetENP's enzyme_gene_info function retrieves genes and enzymes involved in the associated KEGG pathways, and react_substrate reports whether each metabolite is a reactant or product: [evidence span withheld — anchor status: not_found]
- [results] 77 reactions, 109 genes, and 46 enzymes were associated to the significantly altered metabolites: [evidence span withheld — anchor status: not_found]
