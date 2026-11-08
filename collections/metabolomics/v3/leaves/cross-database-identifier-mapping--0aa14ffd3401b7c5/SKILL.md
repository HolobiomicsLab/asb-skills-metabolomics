---
name: cross-database-identifier-mapping
description: Use when integrating pathway/annotation records from HMDB XML, KEGG REST
  responses, Reactome ChEBI/UniProt mappings, or WikiPathways GPML files whose metabolite
  and gene identifiers use different, overlapping schemes.
when_to_use_negative:
- You only need to query an already-built RaMP database — use the RaMP R package (connectToRaMP)
  or the RaMP-DB SQLite bundle instead of rebuilding the mapping pipeline
- Your sources share a single identifier scheme with no synonyms to unify — a straight
  join/load suffices
- You are using RaMP-DB 3.0's ncats backend, where the pipeline and sources (Lipid-MAPS,
  ChEBI, PFOCR, RefMet, Rhea added) differ from the 2018 Mathelab scripts
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RaMP
  role: 'Target schema: the unification procedure assigns shared RaMP IDs to analytes
    found across the four source databases'
  repo: https://github.com/ncats/RaMP-DB/
- name: MySQL
  role: Relational store into which the unified analyte, pathway, and analyte-pathway
    relations are loaded and exported as a dump
- name: RMySQL
  role: R client library used by the RaMP package to connect to the populated database
    after the dump is reloaded
- name: RaMP-BackEnd
  role: Python 3.5 ingestion codebase that runs main.py to generate the SQL files
    implementing the ID dictionaries and synonym merge
  repo: https://github.com/Mathelab/RaMP-BackEnd
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:811267df97072f58c9c42a6294f4cbb6c7eca886ba44e8e81be16223f7995ec3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncats/RaMP-DB
  tools:
  - DBI
  - MySQL
  - RMySQL
  - RaMP
  - RaMP-BackEnd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# cross-database-identifier-mapping

## Summary

A two-step identifier unification procedure from the RaMP backend: build per-source dictionaries of analyte IDs (HMDB, KEGG, Reactome, WikiPathways), then merge synonyms so analytes linked to common IDs share one internal RaMP ID before loading into MySQL.

## When to use

Use when integrating pathway/annotation records from HMDB XML, KEGG REST responses, Reactome ChEBI/UniProt mappings, or WikiPathways GPML files whose metabolite and gene identifiers use different, overlapping schemes.

## When NOT to use

- You only need to query an already-built RaMP database — use the RaMP R package (connectToRaMP) or the RaMP-DB SQLite bundle instead of rebuilding the mapping pipeline
- Your sources share a single identifier scheme with no synonyms to unify — a straight join/load suffices
- You are using RaMP-DB 3.0's ncats backend, where the pipeline and sources (Lipid-MAPS, ChEBI, PFOCR, RefMet, Rhea added) differ from the 2018 Mathelab scripts

## Inputs

- HMDB metabolite XML files
- KEGG pathway data retrieved via the KEGG REST API
- Reactome ChEBI/UniProt mapping files
- WikiPathways GPML files
- Empty MySQL database (e.g. 'ramp')

## Outputs

- Unified analyte, pathway, and analyte-pathway relations in a MySQL relational schema
- RaMP IDs that collapse cross-source synonyms of the same gene or metabolite
- mysql dump file loadable elsewhere and queryable through the RaMP R package

## How to apply

Parse each source-specific format (HMDB metabolite XML, KEGG REST API responses, Reactome ChEBI/UniProt mapping tables, WikiPathways GPML XML) into candidate analyte-to-pathway relations using the RaMP-BackEnd Python pipeline. In step one, construct dictionaries of IDs for each database source; in step two, ensure identifiers linked to common IDs are assigned the same internal RaMP ID, collapsing synonym records so each gene or metabolite is represented once regardless of source scheme. Load the unified analyte, pathway, and analyte-pathway relations into a MySQL relational schema, then export a mysql dump file for redistribution. Correctness hinges on synonym disambiguation: ambiguous synonyms must not collapse distinct analytes into one RaMP ID.

## Related tools

- **RaMP** (Target schema: the unification procedure assigns shared RaMP IDs to analytes found across the four source databases) — https://github.com/ncats/RaMP-DB/
- **MySQL** (Relational store into which the unified analyte, pathway, and analyte-pathway relations are loaded and exported as a dump)
- **RMySQL** (R client library used by the RaMP package to connect to the populated database after the dump is reloaded)
- **RaMP-BackEnd** (Python 3.5 ingestion codebase that runs main.py to generate the SQL files implementing the ID dictionaries and synonym merge) — https://github.com/Mathelab/RaMP-BackEnd

## Examples

```
git clone https://github.com/Mathelab/RaMP-BackEnd && python main.py  # writes SQL files to misc/sql for import into an empty 'ramp' MySQL database
```

## Evaluation signals

- main.py runs to completion and writes SQL files into misc/sql before import via the mySQLBulkCsharp Program.cs bulk loader
- The mysql dump reloads into a fresh MySQL instance without schema or foreign-key errors
- Batch RaMP queries (getPathwayFromAnalyte / rampFastPathFromSource) return pathway memberships for both genes and metabolites against the reloaded database
- Spot-check analytes known to appear in multiple sources (e.g. a metabolite with HMDB and KEGG IDs) resolve to a single RaMP ID, not duplicated rows per source scheme

## Limitations

- Synonyms can be ambiguous, so a naive synonym merge can wrongly unify distinct metabolites or genes; the common-ID linkage step must be validated
- Each source uses different ID schemes and update cadences — the paper notes updates are only 'checked multiple times a year'
- The backend scripts require Python 3.5 specifically, and on macOS Python 2.7 must keep running for essential OS functions
- The Mathelab/RaMP-DB repo is no longer maintained; current development moved to ncats/RaMP-DB, whose 3.0 backend covers expanded sources and ID prefixes

## Evidence

- [methods] RaMP maps identifiers across source databases via a two-step procedure: it first builds dictionaries of IDs for each database source, then ensures that identifiers linked to common IDs are assigned the same internal RaMP ID.: [evidence span withheld — anchor status: not_found]
- [abstract] we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the: "we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the"
- [readme] RaMP is a conglomerate of 4 different databases (Kegg, Reactome, HMDB, and WikiPathways), which includes pathways and annotations for genes and metabolites.: [evidence span withheld — anchor status: external_doc]
- [readme] When gene/protein or metabolite ids are input for queries, IDs should be prepended with their database of origin, e.g. kegg:C02712, hmdb:HMDB04824, or CAS:2566-39-4.: [evidence span withheld — anchor status: external_doc]
