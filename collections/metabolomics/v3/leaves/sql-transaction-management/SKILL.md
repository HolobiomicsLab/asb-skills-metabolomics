---
name: sql-transaction-management
description: Use when building a local PostgreSQL cache database from HMDB/ChEBI/LIPID
  MAPS dump files via bulk insertion, or when storing fetched KEGG/PubChem records
  for later reuse without re-querying APIs.
when_to_use_negative:
- When the metabolite identifiers are already fully present in the local cache and
  db.query can answer them — re-fetching or rebuilding would be wasted work
- When you need only one-off identifier mapping without persistent storage — MetaFetcheR's
  queue-based mapping may suffice without rebuilding the cache from dump files
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_3314
tools:
- name: MetaFetcheR
  role: Provides the bulk_insert_hmdb/chebi/lipidmaps, bulk_insert_secondary_ids,
    db.transaction/db.commit and db.write_df functions used to build and write the
    PostgreSQL cache
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: PostgreSQL
  role: Backend database acting as the cache memory for dump-file bulk loads and on-the-fly
    KEGG/PubChem records
- name: DBI
  role: R database interface layer used by db.connect and db.write_df against PostgreSQL
- name: RPostgreSQL/RPostgres
  role: PostgreSQL drivers for the R connection used in transactional batched writes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:5da76c953c523752f4a521f209615a332d3d4b8269916f8b4a62b68a2291aa78
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/komorowskilab/MetaFetcheR
  repo_url: https://github.com/komorowskilab/MetaFetcheR
  tools:
  - DBI
  - MetaFetcheR
  - PostgreSQL
  - R
  - RPostgreSQL
  - RPostgreSQL/RPostgres
  - RPostgres
  - XML
  - config
  - httr
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# sql-transaction-management

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs and maintains a local PostgreSQL database as a cache layer for MetaFetcheR: bulk-loads HMDB, ChEBI and LIPID MAPS dump files inside transactional writes and persists on-the-fetched KEGG/PubChem records so repeated identifier-mapping queries avoid HTTP calls.

## When to use

Use when building a local PostgreSQL cache database from HMDB/ChEBI/LIPID MAPS dump files via bulk insertion, or when storing fetched KEGG/PubChem records for later reuse without re-querying APIs.

## When NOT to use

- When the metabolite identifiers are already fully present in the local cache and db.query can answer them — re-fetching or rebuilding would be wasted work
- When you need only one-off identifier mapping without persistent storage — MetaFetcheR's queue-based mapping may suffice without rebuilding the cache from dump files

## Inputs

- HMDB database dump file (downloaded from the database provider)
- ChEBI database dump file
- LIPID MAPS dump file
- Secondary/cross-reference identifiers file
- PostgreSQL connection configuration (host, port, db_name, user, password, path) from write_config
- Parsed KEGG and PubChem records produced by http_call_api and create_kegg_record/create_pubchem_record

## Outputs

- Populated local PostgreSQL cache database with tables for HMDB, ChEBI, LIPID MAPS and secondary identifiers
- Cached KEGG and PubChem records persisted in the PostgreSQL tables
- Array-formatted (convert_df_to_db_array) column values written via db.write_df

## How to apply

Write connection settings (host, port, db_name, user, password, path) and dump-file names with write_config(conf), then create the empty schema with db.create_database(conf) and open a connection via db.connect(conf). Wrap batched writes in db.transaction(conf)/db.commit(conf) so bulk_insert_hmdb(filepath), bulk_insert_chebi(filepath), bulk_insert_lipidmaps(filepath) and bulk_insert_secondary_ids(filepath) either fully succeed or roll back. Vector-valued columns must be reformatted with convert_df_to_db_array(df, cvectors) before being written through db.write_df(table, df), because PostgreSQL expects DB-array text rather than R vectors. Once the static databases are loaded, KEGG and PubChem records fetched on the fly (http_call_api(url, db_id) after id_to_url(db_id, db_tag), parsed by create_kegg_record/create_pubchem_record) are inserted into the same PostgreSQL tables, which is the rationale: the local database acts as a cache memory so subsequent queries are answered locally instead of via HTTP, avoiding timeouts caused by excessive calls.

## Related tools

- **MetaFetcheR** (Provides the bulk_insert_hmdb/chebi/lipidmaps, bulk_insert_secondary_ids, db.transaction/db.commit and db.write_df functions used to build and write the PostgreSQL cache) — https://github.com/komorowskilab/MetaFetcheR/
- **PostgreSQL** (Backend database acting as the cache memory for dump-file bulk loads and on-the-fly KEGG/PubChem records)
- **DBI** (R database interface layer used by db.connect and db.write_df against PostgreSQL)
- **RPostgreSQL/RPostgres** (PostgreSQL drivers for the R connection used in transactional batched writes)

## Examples

```
conf <- write_config(host, port, db_name, user, password, path); db.create_database(conf); con <- db.connect(conf); db.transaction(con); bulk_insert_hmdb(filepath); db.commit(con)
```

## Evaluation signals

- db.query(SQL) against the cache returns records for all five databases (HMDB, ChEBI, KEGG, PubChem, LIPID MAPS)
- Repeated KEGG/PubChem identifier queries are served from the PostgreSQL cache with no new HTTP calls issued
- Each db.transaction/db.commit cycle leaves the schema consistent (no partial batch rows) after bulk_insert_hmdb/chebi/lipidmaps runs
- Vector columns written via db.write_df exist as DB arrays (i.e., convert_df_to_db_array was applied) rather than malformed text
- bulk_insert_secondary_ids populated cross-reference rows so queue-based mapping can fill empty identifier fields

## Limitations

- Excessive direct HTTP calls to KEGG/PubChem before the cache is populated can cause timeouts — the cache must be built before heavy querying
- The article reports no version number or changelog for MetaFetcheR, so function signatures (e.g., write_config fields) may drift
- Dump files must be downloaded from HMDB, ChEBI and LIPID MAPS separately; MetaFetcheR only performs the bulk insertion

## Evidence
