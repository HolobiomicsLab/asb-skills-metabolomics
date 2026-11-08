---
name: cross-database-identifier-table-population
description: Use when you need a local cross-database identifier cache for metabolomics
  ID mapping (HMDB, ChEBI, LIPID MAPS, KEGG, PubChem) and repeated lookups must avoid
  live HTTP calls to KEGG/PubChem.
when_to_use_negative:
- You only need a one-off identifier lookup without a persistent local cache and a
  local PostgreSQL instance is not available or writable
- The metabolites come only from a database outside the five supported sources (e.g.
  METLIN, which the article lists as a target database but the README does not list
  as resolvable by MetaFetcheR)
- Input is not metabolite identifiers/dump files but already-mapped identifier tables
  — the queue-based mapping step, not cache population, is what is needed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3307
tools:
- name: MetaFetcheR
  role: R package providing write_config, db.create_database, db.connect, bulk_insert_hmdb/chebi/lipidmaps,
    bulk_insert_secondary_ids, id_to_url, http_call_api and create_kegg_record/create_pubchem_record
    for building the cache
  repo: github.com/komorowskilab/MetaFetcheR
- name: PostgreSQL
  role: Local database acting as the cache memory where dump-file records and on-the-fly
    KEGG/PubChem instances are stored
- name: R
  role: Implementation language for the package and its bulk-insertion and HTTP-caching
    functions
- name: DBI
  role: Dependency for the PostgreSQL connection and table-writing functions (db.connect,
    db.write_df)
- name: httr
  role: Dependency backing the on-the-fly HTTP calls to KEGG and PubChem via http_call_api(url,
    db_id)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:8b4ef34ab800235c892146885e76dc22a530c9f9f2e1fa54ed7fe0b3b4b346ed
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/komorowskilab/MetaFetcheR
  tools:
  - DBI
  - MetaFetcheR
  - PostgreSQL
  - R
  - RPostgreSQL
  - RPostgres
  - XML
  - config
  - httr
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cross-database-identifier-table-population

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs MetaFetcheR's local PostgreSQL cache that links metabolite identifiers across HMDB, ChEBI, LIPID MAPS, KEGG and PubChem: dump files are bulk-inserted into the database, while KEGG and PubChem records are fetched via HTTP on the fly and persisted so repeated queries avoid unnecessary HTTP calls and timeouts.

## When to use

Use when you need a local cross-database identifier cache for metabolomics ID mapping (HMDB, ChEBI, LIPID MAPS, KEGG, PubChem) and repeated lookups must avoid live HTTP calls to KEGG/PubChem.

## When NOT to use

- You only need a one-off identifier lookup without a persistent local cache and a local PostgreSQL instance is not available or writable
- The metabolites come only from a database outside the five supported sources (e.g. METLIN, which the article lists as a target database but the README does not list as resolvable by MetaFetcheR)
- Input is not metabolite identifiers/dump files but already-mapped identifier tables — the queue-based mapping step, not cache population, is what is needed

## Inputs

- HMDB database dump file (filepath passed to bulk_insert_hmdb)
- ChEBI database dump file (filepath passed to bulk_insert_chebi)
- LIPID MAPS database dump file (filepath passed to bulk_insert_lipidmaps)
- Secondary-ids cross-reference file (filepath passed to bulk_insert_secondary_ids)
- Connection configuration (host, port, db_name, user, password, path) written via write_config(...)
- KEGG and PubChem identifiers to fetch on the fly (converted to URLs by id_to_url(db_id, db_tag))

## Outputs

- Populated local PostgreSQL cache database containing records for HMDB, ChEBI, LIPID MAPS, KEGG and PubChem
- Cached KEGG records (created by create_kegg_record) and PubChem records (created by create_pubchem_record) persisted in PostgreSQL tables
- Cross-reference secondary identifier tables inserted via bulk_insert_secondary_ids
- Dataframes converted to PostgreSQL array format via convert_df_to_db_array before writing with db.write_df(table, df)

## How to apply

Install MetaFetcheR from GitHub with install_github("shizidushu/hfun","komorowskilab/metafetcher") and load DBI/RPostgreSQL/RPostgres, httr and XML. Write a config (host, port, db_name, user, password, path, plus HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name) with write_config(...), create the empty database with db.create_database(conf) and open a connection via db.connect(conf), wrapping batched writes in db.transaction/db.commit. Download the HMDB, ChEBI and LIPID MAPS dump files and load them with bulk_insert_hmdb(filepath), bulk_insert_chebi(filepath) and bulk_insert_lipidmaps(filepath); format vector columns with convert_df_to_db_array(df, cvectors), write tables with db.write_df(table, df), and add cross-references via bulk_insert_secondary_ids(filepath). KEGG and PubChem have no dumps, so their records are cached on the fly: build request URLs with id_to_url(db_id, db_tag), fetch with http_call_api(url, db_id), parse into create_kegg_record / create_pubchem_record objects, and persist them into the PostgreSQL tables so subsequent queries for the same identifier are served from the cache instead of over HTTP. The rationale is that the local PostgreSQL database acts as a cache memory of information, resolving the missing and inconsistent foreign reference identifiers across the five small-compound databases.

## Related tools

- **MetaFetcheR** (R package providing write_config, db.create_database, db.connect, bulk_insert_hmdb/chebi/lipidmaps, bulk_insert_secondary_ids, id_to_url, http_call_api and create_kegg_record/create_pubchem_record for building the cache) — github.com/komorowskilab/MetaFetcheR
- **PostgreSQL** (Local database acting as the cache memory where dump-file records and on-the-fly KEGG/PubChem instances are stored)
- **R** (Implementation language for the package and its bulk-insertion and HTTP-caching functions)
- **DBI** (Dependency for the PostgreSQL connection and table-writing functions (db.connect, db.write_df))
- **httr** (Dependency backing the on-the-fly HTTP calls to KEGG and PubChem via http_call_api(url, db_id))

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher")
```

## Evaluation signals

- Run db.query(SQL) against the cache and confirm records exist for all five databases (HMDB, ChEBI, LIPID MAPS, KEGG, PubChem)
- Confirm that KEGG and PubChem entries already stored in PostgreSQL are returned from the cache rather than re-fetched over HTTP
- Verify vector-typed columns were converted with convert_df_to_db_array(df, cvectors) before db.write_df(table, df) succeeded without array-format errors
- Confirm secondary identifier cross-reference tables were populated via bulk_insert_secondary_ids(filepath)
- Check that the bulk-inserted dump tables (bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps) contain rows matching the downloaded dump files

## Limitations

- Only five databases are supported (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS); METLIN identifiers from the case studies are not resolvable
- Unique MetaFetcheR identifiers are only a planned future feature, not currently available
- The article does not state a package version number, so cache schema reproducibility depends on the GitHub state at install time

## Evidence
