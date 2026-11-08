---
name: metabolite-gene-annotation-counting
description: Use when you have a RaMP MySQL dump loaded locally and need to quantify
  what fraction of metabolites or genes are annotated by each source database and
  what fraction is shared across all four.
when_to_use_negative:
- The goal is pathway enrichment or Fisher overrepresentation testing on an analyte
  list, rather than characterizing annotation coverage of the database itself (use
  runFisherTest/runCombinedFisherTest instead).
- The backend database being queried is RaMP-DB 3.0 (ncats/RaMP-DB), which integrates
  additional sources (Lipid-MAPS, ChEBI, PFOCR, RefMet, Rhea) beyond the four original
  databases, so the 4-database overlap logic no longer applies.
- Input is not a RaMP MySQL dump but raw source-database files (HMDB, KEGG, Reactome,
  WikiPathways downloads); those must first be built into RaMP via RaMP-BackEnd main.py
  before counting.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: RaMP
  role: R package whose connectToRaMP() opens the MySQL connection used to query analyte-pathway
    mappings
  repo: https://github.com/Mathelab/RaMP-DB
- name: MySQL
  role: Relational database server hosting the imported RaMP mysql dump that is queried
    for annotations
- name: RMySQL
  role: R driver (with DBI) through which the RaMP package executes the analyte-pathway
    SQL queries
- name: dplyr
  role: Used in R to aggregate counts and assemble the coverage table data frame
- name: RaMP-BackEnd
  role: Python backend that created the RaMP database; its fourVenn.R/threeVenn.R
    scripts produce the equivalent overlap plots
  repo: https://github.com/Mathelab/RaMP-BackEnd
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:6d5071efc2c6cd35543f00305683858c8b998d4df2d3876902a96224c48dd639
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# metabolite-gene-annotation-counting

## Summary

Quantifies per-database annotation coverage and cross-database overlap of metabolites and genes integrated in the RaMP relational database (KEGG, Reactome, HMDB, WikiPathways), reproducing the coverage/uniqueness statistics reported in the RaMP paper's Figure 2.

## When to use

Use when you have a RaMP MySQL dump loaded locally and need to quantify what fraction of metabolites or genes are annotated by each source database and what fraction is shared across all four.

## When NOT to use

- The goal is pathway enrichment or Fisher overrepresentation testing on an analyte list, rather than characterizing annotation coverage of the database itself (use runFisherTest/runCombinedFisherTest instead).
- The backend database being queried is RaMP-DB 3.0 (ncats/RaMP-DB), which integrates additional sources (Lipid-MAPS, ChEBI, PFOCR, RefMet, Rhea) beyond the four original databases, so the 4-database overlap logic no longer applies.
- Input is not a RaMP MySQL dump but raw source-database files (HMDB, KEGG, Reactome, WikiPathways downloads); those must first be built into RaMP via RaMP-BackEnd main.py before counting.

## Inputs

- RaMP MySQL dump file (inst/extdata from Mathelab/RaMP-DB) imported into a local MySQL server database
- MySQL connection parameters (dbname, username, conpass, host)
- RaMP analyte-pathway mapping tables with source-database labels and analyte type (metabolite vs gene)

## Outputs

- 8-row coverage table (4 source databases x 2 analyte types) of per-database uniqueness percentages, written to coverage_table.csv
- Two all-four-database overlap percentages: one for metabolites (~0.05%) and one for genes (~13.2%)

## How to apply

Load the raw RaMP MySQL dump (from RaMP-DB inst/extdata) into a local MySQL server and open a connection with the RaMP R package's connectToRaMP(dbname, username, conpass, host) via RMySQL/DBI. Query the analyte-pathway mapping tables to retrieve all analyte-to-pathway annotations, retaining the source-database label (KEGG, Reactome, HMDB, WikiPathways) and the analyte type (metabolite vs gene). Separately for metabolites and genes, count analytes annotated by exactly one source database and divide by the total annotated analytes of that type, yielding 4 databases x 2 analyte types = 8 uniqueness percentages. Count analytes annotated in all four source databases simultaneously and express as a percentage of total metabolites and total genes. Assemble the 8-row coverage table plus the two all-four-overlap percentages into a single data frame with dplyr and write to coverage_table.csv.

## Related tools

- **RaMP** (R package whose connectToRaMP() opens the MySQL connection used to query analyte-pathway mappings) — https://github.com/Mathelab/RaMP-DB
- **MySQL** (Relational database server hosting the imported RaMP mysql dump that is queried for annotations)
- **RMySQL** (R driver (with DBI) through which the RaMP package executes the analyte-pathway SQL queries)
- **dplyr** (Used in R to aggregate counts and assemble the coverage table data frame)
- **RaMP-BackEnd** (Python backend that created the RaMP database; its fourVenn.R/threeVenn.R scripts produce the equivalent overlap plots) — https://github.com/Mathelab/RaMP-BackEnd

## Examples

```
con <- RaMP::connectToRaMP(dbname='ramp', username='root', conpass='mypassword', host='localhost')
```

## Evaluation signals

- coverage_table.csv contains exactly 8 rows, one per source database x analyte type (metabolite, gene).
- Every per-database uniqueness percentage lies in the interval (0, 100].
- The all-four-database overlap reproduces ~0.05% for metabolites and ~13.2% for genes, matching Figure 2 of the RaMP paper.
- Total annotated analyte counts per type equal the sum of unique-per-database and multi-database-annotated analytes (no double counting).
- Each source label in the output is restricted to {KEGG, Reactome, HMDB, WikiPathways} and each analyte type to {metabolite, gene}.

## Limitations

- The Mathelab/RaMP-DB repository is no longer maintained; users are directed to ncats/RaMP-DB, whose 3.0 backend integrates additional sources beyond the original four, changing overlap semantics.
- The paper's figures reflect a 2018 snapshot of KEGG, Reactome, HMDB, and WikiPathways; reproducing exact percentages requires the matching mysql dump version.
- Only analytes with at least one pathway annotation are counted; unannotated metabolites/genes are excluded from the denominators.

## Evidence

- [abstract] four source databases integrated into RaMP: "biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the Human Metabolome DataBase (HMDB)"
- [methods] connection function: "connectToRaMP(dbname, username, conpass, host)"
- [abstract] mysql dump availability: "The package also includes the raw database file (mysql dump), thereby providing a stand-alone downloadable framework for public use"
- [readme] overlap plots in backend: [evidence span withheld — anchor status: external_doc]
- [readme] database build/import workflow: [evidence span withheld — anchor status: external_doc]
