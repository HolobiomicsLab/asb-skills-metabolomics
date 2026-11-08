---
name: database-coverage-quantification
description: Use when you have an integrated multi-source annotation database and
  need per-database annotation coverage and cross-source overlap statistics for metabolites
  vs. genes.
when_to_use_negative:
- Your goal is pathway over-representation/enrichment testing rather than characterizing
  database annotation coverage — use runFisherTest / runCombinedFisherTest instead.
- You need up-to-date annotations from the expanded RaMP-DB 3.0 sources (ChEBI, Lipid-MAPS,
  RefMet, Rhea, PFOCR) rather than reproducing the original four-database coverage
  analysis.
- The analytes you care about are not mapped into the integrated database's analyte-pathway
  tables (e.g., you only have raw source files and no MySQL dump loaded).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3172
tools:
- name: RaMP
  role: R package providing connectToRaMP() to open the MySQL connection and access
    the analyte-pathway mapping tables
  repo: https://github.com/Mathelab/RaMP-DB
- name: RMySQL
  role: R/DBI driver used by connectToRaMP to query the RaMP MySQL database
- name: dplyr
  role: R data manipulation used to assemble the 8-row coverage table and overlap
    percentages into a single data frame
- name: MySQL
  role: Hosts the raw RaMP mysql dump that must be imported before querying coverage
    statistics
- name: RaMP-BackEnd
  role: Python pipeline (main.py, mySQLBulkCsharp) that builds the RaMP database and
    includes threeVenn.R/fourVenn.R overlap plots
  repo: https://github.com/Mathelab/RaMP-BackEnd
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:2466a4ee5b7da97bd2d22274d40587b6c077d4620bfc6138ee8dd52d0009e848
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - MySQL
  - R
  - RMySQL
  - RaMP
  - RaMP-BackEnd
  - dplyr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# database-coverage-quantification

## Summary

Quantifies how completely each source database (KEGG, Reactome, HMDB, WikiPathways) annotates metabolites and genes within an integrated pathway database like RaMP, and computes cross-database overlap including the fraction of analytes annotated by all four sources.

## When to use

Use when you have an integrated multi-source annotation database and need per-database annotation coverage and cross-source overlap statistics for metabolites vs. genes.

## When NOT to use

- Your goal is pathway over-representation/enrichment testing rather than characterizing database annotation coverage — use runFisherTest / runCombinedFisherTest instead.
- You need up-to-date annotations from the expanded RaMP-DB 3.0 sources (ChEBI, Lipid-MAPS, RefMet, Rhea, PFOCR) rather than reproducing the original four-database coverage analysis.
- The analytes you care about are not mapped into the integrated database's analyte-pathway tables (e.g., you only have raw source files and no MySQL dump loaded).

## Inputs

- RaMP MySQL dump (mysql dump from github.com/Mathelab/RaMP-DB/tree/master/inst/extdata) loaded into a local MySQL server
- MySQL connection credentials (dbname, username, conpass, host)
- Analyte-pathway mapping tables with source-database labels (KEGG, Reactome, HMDB, WikiPathways) and analyte type (metabolite vs gene)

## Outputs

- coverage_table.csv: 8 rows (one per source database x analyte type) with per-database uniqueness percentages
- All-four-overlap percentages: fraction of metabolites (~0.05%) and genes (~13.2%) shared across all four source databases
- Optional Venn diagrams of overlapping genes and metabolites across the four databases (threeVenn.R / fourVenn.R)

## How to apply

Load the raw RaMP MySQL dump into a local MySQL server and open a connection with connectToRaMP(dbname, username, conpass, host) via RMySQL/DBI. Query the analyte-pathway mapping tables so each mapping retains its source-database label (KEGG, Reactome, HMDB, WikiPathways) and analyte type (metabolite vs gene). Count, separately for metabolites and genes, analytes annotated by exactly one source database and divide by the total annotated analytes of that type, producing a 4 databases x 2 analyte types = 8-row coverage table. Then count analytes annotated in all four source databases simultaneously and express them as a percentage of total metabolites and total genes to obtain the all-four-overlap figures. Assemble the 8 rows plus the two overlap percentages into a single data frame with dplyr and write it to coverage_table.csv; the README notes that R scripts (src/threeVenn.R and fourVenn.R, using VennDiagram and grDevices) produce the corresponding overlap plots.

## Related tools

- **RaMP** (R package providing connectToRaMP() to open the MySQL connection and access the analyte-pathway mapping tables) — https://github.com/Mathelab/RaMP-DB
- **RMySQL** (R/DBI driver used by connectToRaMP to query the RaMP MySQL database)
- **dplyr** (R data manipulation used to assemble the 8-row coverage table and overlap percentages into a single data frame)
- **MySQL** (Hosts the raw RaMP mysql dump that must be imported before querying coverage statistics)
- **RaMP-BackEnd** (Python pipeline (main.py, mySQLBulkCsharp) that builds the RaMP database and includes threeVenn.R/fourVenn.R overlap plots) — https://github.com/Mathelab/RaMP-BackEnd

## Examples

```
con <- RaMP::connectToRaMP(dbname='ramp', username='root', conpass='***', host='localhost')
```

## Evaluation signals

- coverage_table.csv contains exactly 8 rows, one per source database x analyte type
- Every per-database uniqueness percentage lies in (0, 100]
- All-four-overlap reproduces ~0.05% for metabolites and ~13.2% for genes as reported for Figure 2
- Each row's percentage is computed against the total annotated analytes of that type (metabolite or gene), not a pooled total
- Every analyte-pathway mapping counted carries exactly one valid source-database label from {KEGG, Reactome, HMDB, WikiPathways}

## Limitations

- The original Mathelab/RaMP-DB repo is no longer maintained; use https://github.com/ncats/RaMP-DB/, and RaMP-DB 3.0 draws from expanded sources (HMDB, KEGG, Lipid-MAPS, WikiPathways, Reactome, CheBI, PFOCR, RefMet, Rhea), so the four-database overlap figures will not reproduce on the new backend.
- Coverage statistics reflect the snapshot of source databases integrated at RaMP build time; source updates are checked only multiple times a year.
- Running main.py with getDatabaseFiles=True is needed on first build because source data folders are not included in the repo due to size restrictions.

## Evidence

- [results] Only a very small fraction of analytes is shared across all four source databases integrated into RaMP: 0.05% of metabolites and 13.2% of genes overlap between all four databases.: [evidence span withheld — anchor status: not_found]
- [methods] connectToRaMP function signature from methods: "connectToRaMP(dbname, username, conpass, host)"
- [abstract] Source databases integrated into RaMP: "biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the Human Metabolome DataBase (HMDB)"
- [readme] README on Venn scripts: [evidence span withheld — anchor status: external_doc]
- [readme] README on the mysql dump location: [evidence span withheld — anchor status: external_doc]
- [readme] README on repo maintenance status: "This repo is no longer maintained.  Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
