---
name: cross-database-analyte-overlap-analysis
description: 'Use when evaluating annotation coverage of an integrated pathway database:
  you need to compute per-source-database uniqueness and multi-database overlap percentages
  for metabolite and gene analytes, e.g. to reproduce Figure-2-style coverage statistics
  from the RaMP MySQL dump.'
when_to_use_negative:
- When the goal is pathway or chemical enrichment testing rather than coverage statistics
  — use RaMP's runFisherTest / runCombinedFisherTest / FilterFishersResults pipeline
  instead
- When working with RaMP-DB 3.0, which draws annotations from more than four sources
  (HMDB, KEGG via HMDB, Lipid-MAPS, WikiPathways, Reactome, CheBI, PFOCR, RefMet,
  Rhea) — the four-database overlap framing and 0.05%/13.2% figures no longer apply
- When analyte identifiers have not been reconciled/synonym-mapped across source databases
  — unmatched IDs would silently deflate cross-database overlap counts
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: RaMP
  role: R package providing connectToRaMP() to open the MySQL connection and the analyte-pathway
    mapping tables queried for overlap counts
  repo: https://github.com/Mathelab/RaMP-DB
- name: RMySQL
  role: R database driver (imported by RaMP) used with DBI to execute the analyte-pathway
    mapping queries
- name: dplyr
  role: Used to aggregate counts by source database and analyte type and assemble
    the 8-row coverage data frame for export
- name: VennDiagram
  role: R package used by RaMP-BackEnd's threeVenn.R and fourVenn.R to visualize gene
    and metabolite overlap across the source databases
  repo: https://github.com/Mathelab/RaMP-BackEnd
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:42c0cc590707c132fd6b0db5b44c14b9338e3974cbf01f8b4cd88fcb6b605f96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - R
  - RMySQL
  - RaMP
  - VennDiagram
  - dplyr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# cross-database-analyte-overlap-analysis

## Summary

Quantifies how many metabolites and genes in the RaMP relational pathway database are annotated by exactly one versus multiple source databases (KEGG, Reactome, HMDB, WikiPathways), including the fraction shared across all four. This exposes the annotation coverage gaps that motivate RaMP's integrated, multi-source design.

## When to use

Use when evaluating annotation coverage of an integrated pathway database: you need to compute per-source-database uniqueness and multi-database overlap percentages for metabolite and gene analytes, e.g. to reproduce Figure-2-style coverage statistics from the RaMP MySQL dump.

## When NOT to use

- When the goal is pathway or chemical enrichment testing rather than coverage statistics — use RaMP's runFisherTest / runCombinedFisherTest / FilterFishersResults pipeline instead
- When working with RaMP-DB 3.0, which draws annotations from more than four sources (HMDB, KEGG via HMDB, Lipid-MAPS, WikiPathways, Reactome, CheBI, PFOCR, RefMet, Rhea) — the four-database overlap framing and 0.05%/13.2% figures no longer apply
- When analyte identifiers have not been reconciled/synonym-mapped across source databases — unmatched IDs would silently deflate cross-database overlap counts

## Inputs

- RaMP MySQL dump file (inst/extdata of Mathelab/RaMP-DB) loaded into a local MySQL server
- Analyte-pathway mapping tables with per-mapping source-database labels (KEGG, Reactome, HMDB, WikiPathways)
- Analyte type labels distinguishing metabolites from genes/proteins

## Outputs

- coverage_table.csv — 8-row data frame (one row per source database x analyte type) of per-database uniqueness percentages (dplyr)
- Two all-four-database overlap percentages: fraction of metabolites and fraction of genes annotated by KEGG, Reactome, HMDB, and WikiPathways simultaneously
- Optional Venn diagrams of gene and metabolite overlap across the four databases (threeVenn.R / fourVenn.R)

## How to apply

Load the raw RaMP MySQL dump (from Mathelab/RaMP-DB inst/extdata) into a local MySQL server and open a connection with the RaMP R package's connectToRaMP(dbname, username, conpass, host) via RMySQL/DBI. Query the analyte-pathway mapping tables to retrieve all analytes annotated to pathways, retaining the source-database label (KEGG, Reactome, HMDB, WikiPathways) and the analyte type (metabolite vs gene) for every mapping. Separately for metabolites and genes, count analytes annotated by exactly one source database and divide by the total annotated analytes of that type to get per-database uniqueness percentages (4 databases x 2 analyte types = 8 rows). Then count analytes annotated in all four source databases simultaneously and express each as a percentage of total metabolites and total genes. Assemble the 8-row table plus the two overlap percentages into a data frame with dplyr and export to coverage_table.csv; optionally cross-check overlap structure with the RaMP-BackEnd R scripts threeVenn.R / fourVenn.R (VennDiagram, grDevices). The rationale: overlap statistics justify RaMP's value proposition — no single source database covers both analyte types comprehensively.

## Related tools

- **RaMP** (R package providing connectToRaMP() to open the MySQL connection and the analyte-pathway mapping tables queried for overlap counts) — https://github.com/Mathelab/RaMP-DB
- **RMySQL** (R database driver (imported by RaMP) used with DBI to execute the analyte-pathway mapping queries)
- **dplyr** (Used to aggregate counts by source database and analyte type and assemble the 8-row coverage data frame for export)
- **VennDiagram** (R package used by RaMP-BackEnd's threeVenn.R and fourVenn.R to visualize gene and metabolite overlap across the source databases) — https://github.com/Mathelab/RaMP-BackEnd

## Examples

```
con <- connectToRaMP(dbname='ramp', username='root', conpass='mypassword', host='localhost')
```

## Evaluation signals

- The assembled coverage table contains exactly 8 rows (one per source database x analyte type)
- Every per-database uniqueness percentage lies in the interval (0, 100]
- The all-four-database overlap reproduces the published figures: ~0.05% of metabolites and ~13.2% of genes shared across all four databases
- Per-database uniqueness percentages do not sum to 100% within an analyte type, confirming multi-source annotated analytes exist in the residual
- Venn diagrams from threeVenn.R/fourVenn.R qualitatively agree with the computed overlap counts (four-way intersection is tiny for metabolites, substantial for genes)

## Limitations

- The Mathelab/RaMP-DB repository is no longer maintained; current RaMP is at ncats/RaMP-DB and RaMP-DB 3.0 integrates more databases, so overlap figures are version-dependent
- Results depend on the database build snapshot; RaMP updates are checked only multiple times a year, so source database content drifts between builds
- No changelog is available for the source repository, making it hard to tie overlap percentages to a specific database release
- Metabolite overlap is extremely small (~0.05%), so counts of all-four-shared metabolites are sensitive to identifier/synonym mapping choices (getPathwayFromAnalyte/find_synonym handling)

## Evidence

- [abstract] RaMP combines four source databases with gene and metabolite pathway annotations: "we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the"
- [methods] connectToRaMP signature used to open the database connection: "connectToRaMP(dbname, username, conpass, host)"
- [readme] README confirms RaMP is a conglomerate of the four databases with analyte annotations: [evidence span withheld — anchor status: external_doc]
- [readme] README documents dedicated R Venn scripts for cross-database overlap plots: [evidence span withheld — anchor status: external_doc]
