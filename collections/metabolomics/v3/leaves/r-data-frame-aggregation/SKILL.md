---
name: r-data-frame-aggregation
description: Use when a gene or metabolite pathway-annotation table carries a source-database
  label (KEGG, Reactome, HMDB, WikiPathways) and you must quantify how many analytes
  are unique to each database versus shared across all four.
when_to_use_negative:
- The analyte-pathway table lacks a source-database label column, so per-database
  grouping and uniqueness counting cannot be performed
- The goal is pathway over-representation analysis rather than database-coverage statistics
  — use RaMP's runFisherTest/runCombinedFisherTest instead
- Working with RaMP-DB 3.0 (ncats/RaMP-DB), whose expanded annotation sources (ChEBI,
  LipidMAPS, PFOCR, RefMet, Rhea) change the four-database overlap interpretation
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: RaMP
  role: Source relational database of metabolomics pathways; its MySQL dump supplies
    the analyte-pathway mappings being aggregated
  repo: https://github.com/Mathelab/RaMP-DB
- name: RMySQL
  role: R/DBI driver used by connectToRaMP to open the MySQL connection and retrieve
    analyte-pathway mapping rows for aggregation
  repo: https://github.com/Mathelab/RaMP-DB
- name: MySQL
  role: Local server hosting the imported RaMP MySQL dump that is queried for analyte-pathway
    mappings
- name: dplyr
  role: R package used to group, count, and assemble the 8-row coverage table and
    overlap percentages into a single data frame written to coverage_table.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:5f864aafaa15c89ec8b4ae21da04320f01a38afa06513f07d8ef3e5c72aa36a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - MySQL
  - R
  - RMySQL
  - RaMP
  - dplyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# r-data-frame-aggregation

## Summary

Aggregate an analyte-pathway mapping table in R (dplyr) by source-database label and analyte type to compute per-database annotation-coverage percentages and all-four-database overlap statistics for the RaMP metabolomics pathway database.

## When to use

Use when a gene or metabolite pathway-annotation table carries a source-database label (KEGG, Reactome, HMDB, WikiPathways) and you must quantify how many analytes are unique to each database versus shared across all four.

## When NOT to use

- The analyte-pathway table lacks a source-database label column, so per-database grouping and uniqueness counting cannot be performed
- The goal is pathway over-representation analysis rather than database-coverage statistics — use RaMP's runFisherTest/runCombinedFisherTest instead
- Working with RaMP-DB 3.0 (ncats/RaMP-DB), whose expanded annotation sources (ChEBI, LipidMAPS, PFOCR, RefMet, Rhea) change the four-database overlap interpretation

## Inputs

- Raw RaMP MySQL dump (from Mathelab/RaMP-DB inst/extdata) loaded into a local MySQL server
- Analyte-pathway mapping table rows with columns for analyte ID, source-database label (KEGG, Reactome, HMDB, WikiPathways), and analyte type (metabolite vs gene)

## Outputs

- coverage_table.csv containing an 8-row data frame (one row per database x analyte type) of per-database uniqueness percentages plus the two all-four-overlap percentages
- All-four-database overlap percentages for metabolites (~0.05%) and genes (~13.2%)

## How to apply

Load the raw RaMP MySQL dump into a local MySQL server and connect with the RaMP R package's connectToRaMP(dbname, username, conpass, host) using RMySQL/DBI. Query the analyte-pathway mapping tables, retaining the source-database label (KEGG, Reactome, HMDB, WikiPathways) and the analyte type (metabolite vs gene) on every mapping row. Group by analyte type and count analytes annotated by exactly one source database, dividing by the total annotated analytes of that type to obtain per-database uniqueness percentages (4 databases x 2 analyte types = 8 rows). Separately count analytes annotated by all four source databases at once and express each as a percentage of total metabolites and total genes. Assemble the 8-row coverage table and the two overlap percentages into a single data frame with dplyr and export it as coverage_table.csv.

## Related tools

- **RaMP** (Source relational database of metabolomics pathways; its MySQL dump supplies the analyte-pathway mappings being aggregated) — https://github.com/Mathelab/RaMP-DB
- **RMySQL** (R/DBI driver used by connectToRaMP to open the MySQL connection and retrieve analyte-pathway mapping rows for aggregation) — https://github.com/Mathelab/RaMP-DB
- **MySQL** (Local server hosting the imported RaMP MySQL dump that is queried for analyte-pathway mappings)
- **dplyr** (R package used to group, count, and assemble the 8-row coverage table and overlap percentages into a single data frame written to coverage_table.csv)

## Examples

```
con <- connectToRaMP(dbname="ramp", username="user", conpass="pw", host="localhost"); mappings <- dbGetQuery(con, "SELECT analyte, source_db, analyte_type FROM analytepathway")
```

## Evaluation signals

- Resulting coverage table contains exactly 8 rows (4 databases x 2 analyte types)
- Every per-database uniqueness percentage lies in the interval (0, 100]
- All-four-overlap percentages reproduce the reported values: ~0.05% for metabolites and ~13.2% for genes (Figure 2)
- Denominators (total annotated metabolites and total annotated genes) match the row counts returned by the mapping-table queries
- coverage_table.csv parses back into a single data frame containing both the coverage rows and the two overlap values

## Limitations

- The Mathelab/RaMP-DB repository is no longer maintained; users are directed to ncats/RaMP-DB, whose v3.0 backend draws from more sources (HMDB, KEGG through HMDB, Lipid-MAPS, WikiPathways, Reactome, ChEBI, PFOCR, RefMet, Rhea), so the four-database overlap statistics are not directly comparable
- Rebuilding the database from RaMP-BackEnd requires running main.py with getDatabaseFiles=True, then bulk-importing the generated .sql files with a separate C# script (mySQLBulkCsharp/Program.cs)
- The repo's own overlap plots (threeVenn.R, fourVenn.R) require R with the VennDiagram and grDevices packages, adding an environment dependency

## Evidence

- [methods] connectToRaMP signature: "connectToRaMP(dbname, username, conpass, host)"
- [readme] Four source databases integrated: [evidence span withheld — anchor status: external_doc]
- [readme] Overlap plots across databases: [evidence span withheld — anchor status: external_doc]
- [abstract] Raw database file distributed with package: "The package also includes the raw database file (mysql dump), thereby providing a stand-alone downloadable framework for public use"
- [readme] Repository maintenance status: "This repo is no longer maintained.  Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
