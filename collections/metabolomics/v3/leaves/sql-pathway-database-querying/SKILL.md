---
name: sql-pathway-database-querying
description: Use when a task requires programmatic retrieval of pathway annotations
  for metabolites and/or genes from the integrated KEGG, Reactome, HMDB, and WikiPathways
  sources, e.g. computing per-database coverage, cross-database overlap, or feeding
  pathway IDs into Fisher-style enrichment.
when_to_use_negative:
- The input is raw, unannotated mass-spectrometry peak data with no metabolite/gene
  identifiers to map to pathways
- The user only needs a handful of interactive, one-off lookups — the RShiny web app
  supports basic simple and batch queries without local database setup
- The required pathway sources or annotation types (e.g. chemical classes, reactions
  in RaMP-DB 3.0) fall outside the 2.x dump's four-source scope of KEGG, Reactome,
  HMDB, and WikiPathways
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3275
- http://edamontology.org/topic_0091
tools:
- name: RaMP
  role: R package whose connectToRaMP, getPathwayFromAnalyte, rampFastCata, and rampFastPathFromSource
    functions execute the SQL queries against the local database
  repo: https://github.com/Mathelab/RaMP-DB
- name: MySQL
  role: Relational database engine hosting the RaMP dump; the package ships the raw
    mysql dump for a stand-alone local deployment
  repo: https://github.com/Mathelab/RaMP-BackEnd
- name: RMySQL
  role: DBI driver used by connectToRaMP to open the connection to the local MySQL
    server
  repo: https://github.com/Mathelab/RaMP-DB
- name: RaMP-BackEnd
  role: Python/C# pipeline (main.py plus mySQLBulkCsharp/Program.cs) that regenerates
    and bulk-imports the .sql tables if the dump must be rebuilt
  repo: https://github.com/Mathelab/RaMP-BackEnd
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:efac9abcf9c14c3345af903b924f08db6dc67a46d348faae34afd92f9a5216fa
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3275
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sql-pathway-database-querying

## Summary

Load the RaMP MySQL dump into a local MySQL server and query analyte–pathway mappings for genes and metabolites through the RaMP R package's DBI/RMySQL-backed connection functions, enabling batch/complex queries and downstream coverage or enrichment statistics.

## When to use

Use when a task requires programmatic retrieval of pathway annotations for metabolites and/or genes from the integrated KEGG, Reactome, HMDB, and WikiPathways sources, e.g. computing per-database coverage, cross-database overlap, or feeding pathway IDs into Fisher-style enrichment.

## When NOT to use

- The input is raw, unannotated mass-spectrometry peak data with no metabolite/gene identifiers to map to pathways
- The user only needs a handful of interactive, one-off lookups — the RShiny web app supports basic simple and batch queries without local database setup
- The required pathway sources or annotation types (e.g. chemical classes, reactions in RaMP-DB 3.0) fall outside the 2.x dump's four-source scope of KEGG, Reactome, HMDB, and WikiPathways

## Inputs

- RaMP MySQL dump file (inst/extdata) or RaMP-BackEnd-generated .sql files imported into a local MySQL database (e.g. named 'ramp')
- MySQL connection parameters: dbname, username, conpass, host
- List of analyte identifiers (metabolites and/or genes), optionally with NameOrIds switch for name-vs-ID input
- Source-database label filters (KEGG, Reactome, HMDB, WikiPathways)

## Outputs

- Analyte-to-pathway mapping data frame with pathway IDs, analyte type, and source-database label
- Per-database coverage table (8 rows: 4 databases x 2 analyte types) written to coverage_table.csv
- Catalysis network edges from rampFastCata (consumable by plotCataNetwork)
- Pathway ID sets that feed downstream Fisher overrepresentation testing (fishers_results.csv)

## How to apply

First stand up the database: download the RaMP MySQL dump (inst/extdata in Mathelab/RaMP-DB) or rebuild it with the RaMP-BackEnd python scripts (main.py writes .sql files, then the mySQLBulkCsharp Program.cs bulk-imports them into an empty database such as 'ramp'). Open a connection from R with connectToRaMP(dbname, username, conpass, host), which is backed by RMySQL/DBI. Then use the package's query functions against the analyte-pathway mapping tables: getPathwayFromAnalyte(analytes, find_synonym, conpass, host, dbname, username, NameOrIds) maps analyte lists to pathway identifiers (with optional synonym resolution), rampFastCata retrieves catalysis relationships, and rampFastPathFromSource retrieves pathways from a source database ID. Aggregate the returned rows keeping the source-database label (KEGG, Reactome, HMDB, WikiPathways) and analyte type (metabolite vs gene) so per-database counts can be computed with dplyr. Validate that the mapping tables are non-empty, each query row carries its source label, and derived statistics (e.g. ~0.05% metabolite and ~13.2% gene four-database overlap in Figure 2) reproduce.

## Related tools

- **RaMP** (R package whose connectToRaMP, getPathwayFromAnalyte, rampFastCata, and rampFastPathFromSource functions execute the SQL queries against the local database) — https://github.com/Mathelab/RaMP-DB
- **MySQL** (Relational database engine hosting the RaMP dump; the package ships the raw mysql dump for a stand-alone local deployment) — https://github.com/Mathelab/RaMP-BackEnd
- **RMySQL** (DBI driver used by connectToRaMP to open the connection to the local MySQL server) — https://github.com/Mathelab/RaMP-DB
- **RaMP-BackEnd** (Python/C# pipeline (main.py plus mySQLBulkCsharp/Program.cs) that regenerates and bulk-imports the .sql tables if the dump must be rebuilt) — https://github.com/Mathelab/RaMP-BackEnd

## Examples

```
con <- RaMP::connectToRaMP(dbname="ramp", username="myuser", conpass="mypassword", host="localhost"); pws <- RaMP::getPathwayFromAnalyte(analytes, find_synonym=TRUE, conpass="mypassword", host="localhost", dbname="ramp", username="myuser", NameOrIds="ids")
```

## Evaluation signals

- Connection sanity: connectToRaMP returns a usable DBI/RMySQL connection and simple queries on the pathway tables return non-empty result sets
- Coverage-table shape: the per-database uniqueness table contains exactly 8 rows (4 databases x 2 analyte types) and every uniqueness percentage lies in (0, 100]
- Figure 2 reproduction: all-four-database overlap percentages come out at roughly 0.05% for metabolites and 13.2% for genes
- Analyte mapping check: getPathwayFromAnalyte returns pathway IDs for known analytes, and find_synonym=TRUE resolves synonym inputs without dropping rows
- Source labels are preserved on every mapping row so downstream counting can distinguish KEGG, Reactome, HMDB, and WikiPathways assignments

## Limitations

- The original Mathelab/RaMP-DB repo is no longer maintained; users are directed to ncats/RaMP-DB (RaMP-DB 3.0), whose expanded annotation set (>250,000 metabolites, ~34,000 genes/proteins, >122,000 pathways) differs from the 2.x four-source dump
- The workflow requires a local MySQL server and a bulk import step (Program.cs) before any query can run; the dump and generated .sql data are large and excluded from the BackEnd repo by size
- Source database updates are checked multiple times a year, so absolute coverage and overlap statistics can drift between RaMP versions
- Queries feeding enrichment are subject to the runFisherTest/runCombinedFisherTest min_analyte pathway-size filter, so small pathways may be absent from downstream results

## Evidence

- [methods] connectToRaMP(dbname, username, conpass, host): "connectToRaMP(dbname, username, conpass, host)"
- [abstract] raw database file shipped for stand-alone use: "The package also includes the raw database file (mysql dump), thereby providing a stand-alone downloadable framework for public use"
- [readme] README database setup step: [evidence span withheld — anchor status: external_doc]
- [readme] README bulk import step: [evidence span withheld — anchor status: external_doc]
- [readme] dump location pointer: [evidence span withheld — anchor status: external_doc]
- [abstract] integration scope statement: "combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the"
