---
name: gene-metabolite-pathway-membership-mapping
description: Use when an analyte list contains both genes/proteins and metabolites
  and you need their pathway memberships consolidated from multiple pathway databases
  before enrichment or clustering analysis.
when_to_use_negative:
- The analytes are already annotated with pathway memberships from a single database
  and no cross-database integration is needed
- The goal is pathway enrichment itself rather than membership lookup — use runFisherTest/runCombinedFisherTest
  on top of the mapped memberships instead
- You need organism-specific or non-human pathways outside the HMDB/KEGG/Reactome/WikiPathways
  scope RaMP integrates
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_0602
tools:
- name: RaMP
  role: Relational database and R package performing the gene/metabolite-to-pathway
    membership queries
  repo: https://github.com/Mathelab/RaMP-DB
- name: R
  role: Language of the RaMP R package exposing batch query functions (getPathwayFromAnalyte,
    rampFastPathFromSource)
  repo: https://github.com/Mathelab/RaMP-DB
- name: MySQL
  role: Backend database engine hosting the integrated pathway tables (mysql dump
    distributed with the package)
- name: RShiny
  role: Web application front-end supporting simple and batch membership queries without
    R coding
  repo: https://github.com/Mathelab/RaMP-DB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.3390/metabo8010016
    title: 'RaMP: A Comprehensive Relational Database of Metabolomics Pathways for
      Pathway Enrichment Analysis of Genes and Metabolites'
  source_content_hash: sha256:34cc6bb275f98a8308fdbbb8c99c903e4c8edc9c1e816dcc990c361dc87dab09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathelab/RaMP-DB
  tools:
  - MySQL
  - R
  - RShiny
  - RaMP
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gene-metabolite-pathway-membership-mapping

## Summary

Maps input genes and/or metabolites to biochemical and disease pathways by querying RaMP, a MySQL-backed relational database integrating KEGG, Reactome, WikiPathways, and HMDB, so that mixed analyte lists can be interpreted through a single unified pathway membership resource.

## When to use

Use when an analyte list contains both genes/proteins and metabolites and you need their pathway memberships consolidated from multiple pathway databases before enrichment or clustering analysis.

## When NOT to use

- The analytes are already annotated with pathway memberships from a single database and no cross-database integration is needed
- The goal is pathway enrichment itself rather than membership lookup — use runFisherTest/runCombinedFisherTest on top of the mapped memberships instead
- You need organism-specific or non-human pathways outside the HMDB/KEGG/Reactome/WikiPathways scope RaMP integrates

## Inputs

- List of analyte identifiers (genes/proteins and/or metabolites, e.g. HMDB IDs, gene symbols)
- RaMP MySQL database connection parameters (dbname, username, conpass, host)
- Optional NameOrIds flag and find_synonym flag controlling identifier interpretation and synonym resolution

## Outputs

- Pathway membership table mapping each queried analyte to pathway IDs/names across KEGG, Reactome, WikiPathways, and HMDB
- Pathway-to-analyte membership structures (e.g. pathwaysWithAnalytes) usable as input to compute_overlap_matrix and runFisherTest
- Optional catalysis relationship table for plotCataNetwork

## How to apply

Connect to the RaMP MySQL database using connectToRaMP(dbname, username, conpass, host), then retrieve pathway memberships for the analytes with batch functions such as getPathwayFromAnalyte(analytes, find_synonym, conpass, host, dbname, username, NameOrIds), which optionally resolves synonyms before lookup. Because RaMP integrates KEGG, Reactome, WikiPathways, and HMDB into one relational schema, a single query can return memberships spanning all four sources (e.g. 'list all metabolites involved in glycolysis and lung cancer'). For pathway-side lookups, use rampFastPathFromSource(sourceid, find_synonym, conpass, dbname, username, host) or rampFastCata for catalysis relationships. The resulting pathway-to-analyte membership table feeds downstream steps such as runFisherTest over-representation analysis and compute_overlap_matrix clustering. Validate that every input analyte was either matched (directly or via synonym) or explicitly reported as unmatched, and confirm memberships carry their source database so cross-database redundancy is visible.

## Related tools

- **RaMP** (Relational database and R package performing the gene/metabolite-to-pathway membership queries) — https://github.com/Mathelab/RaMP-DB
- **R** (Language of the RaMP R package exposing batch query functions (getPathwayFromAnalyte, rampFastPathFromSource)) — https://github.com/Mathelab/RaMP-DB
- **MySQL** (Backend database engine hosting the integrated pathway tables (mysql dump distributed with the package))
- **RShiny** (Web application front-end supporting simple and batch membership queries without R coding) — https://github.com/Mathelab/RaMP-DB

## Examples

```
con <- connectToRaMP(dbname='ramp', username='user', conpass='pass', host='localhost'); memberships <- getPathwayFromAnalyte(analytes=c('HMDB0000148','INS','ACAC'), find_synonym=TRUE, conpass='pass', host='localhost', dbname='ramp', username='user', NameOrIds='ids')
```

## Evaluation signals

- Every queried analyte appears in the output either with a pathway membership or an explicit unmatched/synonym-resolution record
- Returned pathway rows carry a source database tag (KEGG, Reactome, WikiPathways, HMDB) so cross-database duplicates can be distinguished
- Batch query results match single-analyte queries run individually for a spot-checked subset of analytes
- Downstream consistency: the pathwaysWithAnalytes structure yields the same membership counts used by runFisherTest and compute_overlap_matrix

## Limitations

- Original Mathelab/RaMP-DB repo is no longer maintained; users are directed to the ncats/RaMP-DB fork (RaMP-DB 3.0) with a different, expanded annotation set (Lipid-MAPS, CheBI, PFOCR, RefMet, Rhea)
- Synonym resolution (find_synonym) may be needed for identifiers in non-canonical form, and unmatched identifiers require manual review
- Cross-database content overlap means the same biological pathway appears multiple times under different database IDs, which motivates downstream clustering

## Evidence

- [abstract] RaMP enables batch and complex queries (e.g., list all metabolites involved in glycolysis and lung cancer), can readily be integrated into pathway analysis tools, and supports pathway: "RaMP enables batch and complex queries (e.g., list all metabolites involved in glycolysis and lung cancer), can readily be integrated into pathway analysis tools, and supports pathway"
- [methods] getPathwayFromAnalyte signature: "getPathwayFromAnalyte(analytes, find_synonym, conpass, host, dbname, username, NameOrIds)"
- [methods] connectToRaMP signature: "connectToRaMP(dbname, username, conpass, host)"
- [abstract] database sources integrated: "biological pathways from the Kyoto Encyclopedia of Genes and Genomes (KEGG), Reactome, WikiPathways, and the Human Metabolome DataBase (HMDB)"
- [readme] deprecated repo notice: "This repo is no longer maintained.  Please go to https://github.com/ncats/RaMP-DB/ to use RaMP."
