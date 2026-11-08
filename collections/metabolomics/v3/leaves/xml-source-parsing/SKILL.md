---
name: xml-source-parsing
description: Use when rebuilding the RaMP database from raw source downloads and each
  source must be parsed from its native format (HMDB XML, KEGG REST output, Reactome
  mapping files, GPML XML) into analyte-pathway relations.
when_to_use_negative:
- You only want to query an existing RaMP database — use the RaMP R package (connectToRaMP)
  instead of re-parsing source files
- You are using RaMP-DB 3.0, which ships a pre-built SQLite file downloaded via the
  R package rather than a MySQL dump rebuilt from XML sources
- Your goal is pathway enrichment analysis (runFisherTest, FilterFishersResults) rather
  than database construction
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3395
tools:
- name: RaMP
  role: Back-end Python pipeline whose parsers convert HMDB XML, KEGG REST, Reactome
    mapping, and GPML inputs into relation tables
  repo: github.com/Mathelab/RaMP-BackEnd
- name: MySQL
  role: Target relational schema into which the parsed and unified analyte/pathway
    relations are bulk-loaded and exported as a mysql dump
- name: RMySQL
  role: R dependency used downstream to connect to the parsed/loaded MySQL database
    for batch queries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:9ebd853b85a4e5fd328af73e080ccc5497ad26881f2f1c13d8eea61fb3af84cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DBI
  - MySQL
  - RMySQL
  - RaMP
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3395
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# xml-source-parsing

## Summary

Parse heterogeneous source-database files (HMDB metabolite XML, KEGG REST responses, Reactome ChEBI/UniProt mapping tables, WikiPathways GPML XML) into candidate analyte-to-pathway relations for the RaMP MySQL knowledgebase. This is the first stage of the RaMP ingestion pipeline that precedes RaMP ID unification and database loading.

## When to use

Use when rebuilding the RaMP database from raw source downloads and each source must be parsed from its native format (HMDB XML, KEGG REST output, Reactome mapping files, GPML XML) into analyte-pathway relations.

## When NOT to use

- You only want to query an existing RaMP database — use the RaMP R package (connectToRaMP) instead of re-parsing source files
- You are using RaMP-DB 3.0, which ships a pre-built SQLite file downloaded via the R package rather than a MySQL dump rebuilt from XML sources
- Your goal is pathway enrichment analysis (runFisherTest, FilterFishersResults) rather than database construction

## Inputs

- HMDB metabolite XML files
- KEGG pathway data retrieved through the KEGG REST API
- Reactome ChEBI/UniProt mapping files
- WikiPathways GPML XML files

## Outputs

- Candidate analyte-to-pathway relation tables per source database
- SQL files generated in misc/sql by main.py for import into MySQL
- Unified RaMP ID mappings covering genes and metabolites across all four sources

## How to apply

Run the RaMP-BackEnd Python codebase (github.com/Mathelab/RaMP-BackEnd), which was built with Python 3.5. Execute main.py so that each source-specific parser converts its native format — HMDB XML, KEGG REST responses, Reactome mapping tables, and WikiPathways GPML XML — into candidate analyte-to-pathway relations. Pass these parsed relations to the ID-unification step, where synonym records across HMDB, KEGG, Reactome, and WikiPathways are merged so identifiers sharing common IDs receive one internal RaMP ID. The pipeline then emits SQL files (e.g. in misc/sql) that are bulk-imported into an empty MySQL database (e.g. 'ramp') and exported as a mysql dump for redistribution.

## Related tools

- **RaMP** (Back-end Python pipeline whose parsers convert HMDB XML, KEGG REST, Reactome mapping, and GPML inputs into relation tables) — github.com/Mathelab/RaMP-BackEnd
- **MySQL** (Target relational schema into which the parsed and unified analyte/pathway relations are bulk-loaded and exported as a mysql dump)
- **RMySQL** (R dependency used downstream to connect to the parsed/loaded MySQL database for batch queries)

## Examples

```
git clone https://github.com/Mathelab/RaMP-BackEnd && python main.py
```

## Evaluation signals

- main.py runs successfully and creates SQL files in the misc/sql folder, as stated in the RaMP-BackEnd README workflow
- The resulting SQL files import cleanly into an empty MySQL database created for the purpose (e.g. named 'ramp')
- The exported mysql dump reloads into another MySQL instance without schema errors
- After reload, batch RaMP queries (getPathwayFromAnalyte) return pathway memberships for both genes and metabolites
- Synonym records linked to common IDs resolve to a single internal RaMP ID across HMDB, KEGG, Reactome, and WikiPathways

## Limitations

- The Mathelab repositories are no longer maintained; README directs users to https://github.com/ncats/RaMP-DB/ and RaMP-DB 3.0 uses SQLite with expanded sources (Lipid-MAPS, ChEBI, PFOCR, RefMet, Rhea)
- RaMP-BackEnd requires Python 3.5; on macOS the preinstalled Python 2.7 must coexist and both be run simultaneously for essential system functions
- Parsing is only the first stage — synonym ambiguity across sources is resolved later by the RaMP ID unification step, so parsed relations alone are not query-ready
- Source database contents change over time; RaMP updates are checked multiple times a year

## Evidence

- [abstract] Multi-source integration scope: "we have developed RaMP (Relational database of Metabolomics Pathways), which combines biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the"
- [abstract] Distributable database artifact: "The package also includes the raw database file (mysql dump), thereby providing a stand-alone downloadable framework for public use"
- [readme] Backend composition: [evidence span withheld — anchor status: external_doc]
- [readme] Pipeline entry point: [evidence span withheld — anchor status: external_doc]
- [readme] Python version requirement: [evidence span withheld — anchor status: external_doc]
