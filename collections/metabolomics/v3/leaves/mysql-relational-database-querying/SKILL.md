---
name: mysql-relational-database-querying
description: Use when an analysis needs structured pathway/analyte membership data
  stored in a MySQL dump (e.g., RaMP's inst/extdata dump) and must be queried programmatically
  in R for batch queries or overlap computations.
when_to_use_negative:
- When the user only needs a single web lookup — the RShiny web app or the RaMP front-end/API
  already supports simple and batch queries without local database administration
- When the target resource is not a relational MySQL database — RaMP's connectToRaMP/RMySQL
  pipeline assumes the mysql dump schema
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: RaMP
  role: R package providing connectToRaMP and query functions (rampFastCata, getPathwayFromAnalyte,
    compute_overlap_matrix, updateOverlapMatrix) that wrap the MySQL database
  repo: https://github.com/Mathelab/RaMP-DB
- name: RMySQL
  role: R driver (Imports) used by RaMP to connect to and query the local MySQL database
- name: DBI
  role: R database interface layer underlying the connectToRaMP connection used for
    all queries
- name: MySQL
  role: Relational database engine hosting the RaMP pathway/analyte tables; RaMP ships
    its raw database as a mysql dump
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:707e09278c17569d1029123912b498ff871c526b742a161cf638387abc056978
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - DBI
  - MySQL
  - R
  - RMySQL
  - RaMP
  - dplyr
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# mysql-relational-database-querying

## Summary

Query a locally installed MySQL relational database (here, the RaMP metabolomics-pathways MySQL dump) through R using RMySQL/DBI, using RaMP's connection and query functions (connectToRaMP, rampFastCata, getPathwayFromAnalyte, compute_overlap_matrix) to retrieve pathway-to-analyte memberships and compute overlap metrics.

## When to use

Use when an analysis needs structured pathway/analyte membership data stored in a MySQL dump (e.g., RaMP's inst/extdata dump) and must be queried programmatically in R for batch queries or overlap computations.

## When NOT to use

- When the user only needs a single web lookup — the RShiny web app or the RaMP front-end/API already supports simple and batch queries without local database administration
- When the target resource is not a relational MySQL database — RaMP's connectToRaMP/RMySQL pipeline assumes the mysql dump schema

## Inputs

- RaMP MySQL database dump (mysql dump file from github.com/Mathelab/RaMP-DB inst/extdata)
- MySQL connection credentials: dbname, username, conpass, host
- List of analyte identifiers or pathway source IDs (e.g., KEGG, Reactome, WikiPathways pathway ids)

## Outputs

- Data frames of pathway-to-metabolite/gene memberships retrieved from RaMP tables
- Pairwise pathway overlap (Jaccard) matrices, e.g., exported to metabolite_overlap_matrix.csv
- Fisher enrichment result data frames (filterable via FilterFishersResults)

## How to apply

First import or load the RaMP MySQL dump (from RaMP-DB inst/extdata, or rebuild it from the RaMP-BackEnd Python scripts by running main.py and bulk-importing the .sql files in misc/sql). Open a connection with RaMP's connectToRaMP(dbname, username, conpass, host), which wraps RMySQL/DBI. Then use RaMP's query functions — rampFastCata, rampFastPathFromSource, getPathwayFromAnalyte, runFisherTest — passing conpass, dbname, username, and host on each call so the same DBI connection serves simple and batch queries. For redundancy-style analyses, pull the pathway-to-metabolite membership tables for KEGG, Reactome, and WikiPathways pathways and feed them to compute_overlap_matrix(pathwayid, pathwaysWithAnalytes, overlapmethod) with a Jaccard overlap metric, then assemble full matrices via updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username), setting together to metabolites-only. Rationale: RaMP was built precisely so pathway data from KEGG, Reactome, WikiPathways, and HMDB 'can readily be integrated into other existing software' as a stand-alone MySQL framework.

## Related tools

- **RaMP** (R package providing connectToRaMP and query functions (rampFastCata, getPathwayFromAnalyte, compute_overlap_matrix, updateOverlapMatrix) that wrap the MySQL database) — https://github.com/Mathelab/RaMP-DB
- **RMySQL** (R driver (Imports) used by RaMP to connect to and query the local MySQL database)
- **DBI** (R database interface layer underlying the connectToRaMP connection used for all queries)
- **MySQL** (Relational database engine hosting the RaMP pathway/analyte tables; RaMP ships its raw database as a mysql dump)

## Examples

```
con <- RaMP::connectToRaMP(dbname="ramp", username="root", conpass=mypassword, host="localhost")
```

## Evaluation signals

- The connectToRaMP call succeeds with the supplied dbname/username/conpass/host and returns a usable DBI connection
- All Jaccard overlap entries from compute_overlap_matrix lie in [0, 1]
- The pairwise metabolite overlap matrix restricted to KEGG-Reactome-WikiPathways reproduces the paper's redundancy pattern: pathways within Reactome and KEGG show the largest overlap
- Batch queries return memberships consistent with known pathway contents (e.g., glycolysis metabolites) across all four source databases (KEGG, Reactome, WikiPathways, HMDB)

## Limitations

- The original Mathelab/RaMP-DB repo is no longer maintained; maintenance moved to ncats/RaMP-DB (RaMP-DB 3.0), so function signatures and schema may differ across versions
- The MySQL dump must be imported locally before any query; RaMP-BackEnd requires python 3.5 and a C# bulk-import step (mySQLBulkCsharp) to build the database from source
- Database content becomes stale — RaMP upstream databases are only checked/updated multiple times a year

## Evidence

- [methods] connectToRaMP with RMySQL/DBI: "connectToRaMP(dbname, username, conpass, host)"
- [abstract] stand-alone downloadable mysql framework: "The package also includes the raw database file (mysql dump), thereby providing a stand-alone downloadable framework for public use"
- [abstract] RaMP data sources: "biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the Human Metabolome DataBase (HMDB)"
- [methods] overlap matrix functions: "updateOverlapMatrix(min_analyte, overlapmethod, together, conpass, host, dbname, username)"
- [readme] README: dump location and rebuild workflow: [evidence span withheld — anchor status: external_doc]
- [readme] README: repo deprecated: "This repo is no longer maintained.  Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
