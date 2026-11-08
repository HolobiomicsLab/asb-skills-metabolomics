---
name: metabolite-identifier-synonym-merging
description: Use when building an integrated pathway database from sources with incompatible
  ID schemes (HMDB XML, KEGG REST, Reactome ChEBI/UniProt maps, WikiPathways GPML)
  and the same analyte must resolve to one record despite synonym ambiguity.
when_to_use_negative:
- When the task is pathway or chemical enrichment analysis on an already-built RaMP
  database (use RaMP's runFisherTest/getPathwayFromAnalyte functions) rather than
  rebuilding or unifying identifiers
- When input identifiers are already uniquely prefixed and no cross-database synonym
  reconciliation is needed (e.g. a single-source list of hmdb:HMDBxxxxx IDs)
- When source databases or species fall outside HMDB/KEGG/Reactome/WikiPathways coverage
  used by RaMP
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: RaMP
  role: Provides the RaMP-BackEnd ingestion code that builds per-source ID dictionaries
    and merges synonyms into internal RaMP IDs
  repo: github.com/Mathelab/RaMP-BackEnd
- name: MySQL
  role: Target relational database into which the unified analyte/pathway/analyte-pathway
    tables are bulk-loaded and from which the dump is exported
- name: RMySQL
  role: R dependency used by the RaMP package to connect to the MySQL database for
    synonym-aware queries (connectToRaMP)
- name: R
  role: Hosts the RaMP package whose query functions consume the unified identifiers
  repo: github.com/Mathelab/RaMP-DB
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:e339755093d05531c0f16c92d84968f60bb3a5376df1621ef2023c36bd405cda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DBI
  - MySQL
  - R
  - RMySQL
  - RaMP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metabolite-identifier-synonym-merging

## Summary

RaMP's two-step procedure for unifying metabolite and gene identifiers that differ across HMDB, KEGG, Reactome, and WikiPathways: build per-source ID dictionaries, then merge synonym records linked by common IDs into a single internal RaMP ID.

## When to use

Use when building an integrated pathway database from sources with incompatible ID schemes (HMDB XML, KEGG REST, Reactome ChEBI/UniProt maps, WikiPathways GPML) and the same analyte must resolve to one record despite synonym ambiguity.

## When NOT to use

- When the task is pathway or chemical enrichment analysis on an already-built RaMP database (use RaMP's runFisherTest/getPathwayFromAnalyte functions) rather than rebuilding or unifying identifiers
- When input identifiers are already uniquely prefixed and no cross-database synonym reconciliation is needed (e.g. a single-source list of hmdb:HMDBxxxxx IDs)
- When source databases or species fall outside HMDB/KEGG/Reactome/WikiPathways coverage used by RaMP

## Inputs

- HMDB metabolite XML files
- KEGG pathway data retrieved via the KEGG REST API
- Reactome ChEBI/UniProt mapping files
- WikiPathways GPML (XML) files
- Source-specific ID dictionaries built by step 1 of the RaMP-BackEnd pipeline

## Outputs

- Unified analyte records where all synonym identifiers map to a single internal RaMP ID
- MySQL relational schema of analyte, pathway, and analyte-pathway relations
- Exported mysql dump file (e.g. inst/extdata in RaMP-DB) reloadable into any MySQL server
- Batch pathway-membership query results for genes and metabolites via the RaMP R package

## How to apply

Run the RaMP-BackEnd Python pipeline (python 3.5, run main.py) to parse each source format — HMDB metabolite XML, KEGG REST API responses, Reactome ChEBI/UniProt mapping tables, and WikiPathways GPML XML — into candidate analyte-to-pathway relations. First build dictionaries of IDs for each database source; this decouples each source's native scheme (hmdb:HMDB04824, kegg:C02712, CAS, uniprot, ensembl, etc.) from the canonical record. Then ensure that identifiers linked to common IDs (shared synonyms) are assigned the same internal RaMP ID, so a gene or metabolite appears once regardless of which source scheme it was queried with; because synonyms can be ambiguous, resolve them through shared-ID linkage rather than name-string equality alone. Load the unified analyte, pathway, and analyte-pathway relations into a MySQL relational schema (empty DB e.g. 'ramp', bulk-imported via the mySQLBulkCsharp Program.cs importer), then export as a mysql dump for reuse. Downstream queries must prepend IDs with their database-of-origin prefix (e.g. kegg:C02712, hmdb:HMDB04824, CAS:2566-39-4) so the synonym tables can map mixed-source input back to RaMP IDs.

## Related tools

- **RaMP** (Provides the RaMP-BackEnd ingestion code that builds per-source ID dictionaries and merges synonyms into internal RaMP IDs) — github.com/Mathelab/RaMP-BackEnd
- **MySQL** (Target relational database into which the unified analyte/pathway/analyte-pathway tables are bulk-loaded and from which the dump is exported)
- **RMySQL** (R dependency used by the RaMP package to connect to the MySQL database for synonym-aware queries (connectToRaMP))
- **R** (Hosts the RaMP package whose query functions consume the unified identifiers) — github.com/Mathelab/RaMP-DB

## Examples

```
Rscript -e 'library(RaMP); rampDB <- RaMP(); RaMP::getPrefixesFromAnalytes(db = rampDB, analyteType = "metabolite")'
```

## Evaluation signals

- Every analyte linked to a shared/common ID across the four sources resolves to exactly one internal RaMP ID (no duplicate records for the same gene or metabolite)
- The exported mysql dump reloads into a MySQL instance and answers batch queries listing pathway memberships for both genes and metabolites
- Mixed-source ID lists (e.g. kegg:C02712 plus hmdb:HMDB04824 plus a CAS number) are all mapped through prefix-aware synonym lookup; RaMP::getPrefixesFromAnalytes returns the expected prefix types for 'metabolite' and 'gene'
- main.py in RaMP-BackEnd runs successfully and produces SQL files in the misc/sql folder that import into an empty database without key/foreign-key errors
- Querying the same analyte by different source identifiers (name vs. HMDB vs. KEGG ID) returns the identical pathway set

## Limitations

- Synonyms can be ambiguous: merging on common IDs may conflate distinct entities or leave ambiguous synonyms unresolved, since each source uses different ID schemes
- The Mathelab/RaMP-DB repo is no longer maintained — active development moved to ncats/RaMP-DB (RaMP-DB 3.0, SQLite-based with expanded annotations)
- RaMP-BackEnd requires Python 3.5 specifically (the README notes coexisting with Python 2.7 on macOS), and the KEGG source depends on its REST API
- Database updates are only checked multiple times a year, so identifier mappings may lag source releases

## Evidence

- [other] RaMP maps identifiers across source databases via a two-step procedure: [evidence span withheld — anchor status: not_found]
- [abstract] we have developed RaMP (Relational database of Metabolomics Pathways): "we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the"
- [methods] connectToRaMP(dbname, username, conpass, host): "connectToRaMP(dbname, username, conpass, host)"
- [readme] RaMP is a conglomerate of 4 different databases: [evidence span withheld — anchor status: external_doc]
- [readme] IDs should be prepended with their database of origin: [evidence span withheld — anchor status: external_doc]
