---
name: metabolite-database-dump-parsing
description: Use when metabolomics identifier mapping requires a local cache across
  HMDB, ChEBI, LIPID MAPS, KEGG and PubChem, and repeated KEGG/PubChem HTTP queries
  must be avoided to prevent timeouts from excessive calls.
when_to_use_negative:
- The identifiers needed are only in databases for which no dump file exists and no
  on-the-fly fetcher is provided (e.g. METLIN is cited in the paper but no bulk-insert
  or HTTP caching path is described for it).
- You only need a one-off single-metabolite lookup — building and maintaining a local
  PostgreSQL cache is overkill compared to direct API queries.
- 'You need unique MetaFetcheR identifiers: the paper states these are a planned future
  feature, not currently available.'
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaFetcheR
  role: R package providing bulk_insert_hmdb/bulk_insert_chebi/bulk_insert_lipidmaps,
    secondary-ID insertion, config/database management (write_config, db.create_database,
    db.connect) and on-the-fly KEGG/PubChem caching functions
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: PostgreSQL
  role: Local database that acts as the cache memory of metabolite information, storing
    bulk-loaded dump data and cached HTTP-fetched KEGG/PubChem records
- name: R
  role: Implementation language; package installed via devtools install_github
- name: DBI / RPostgreSQL / RPostgres
  role: Database connectivity drivers used to open connections and write data frames
    into PostgreSQL tables
- name: httr
  role: HTTP layer for issuing on-the-fly KEGG and PubChem requests via http_call_api(url,
    db_id)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:fde61d06656e194a8bce23fa7b969d3b205b78c1fad24f16316dbd71070e16f4
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
  - DBI / RPostgreSQL / RPostgres
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
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# metabolite-database-dump-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct a local PostgreSQL cache of metabolite identifiers by bulk-loading HMDB, ChEBI and LIPID MAPS database dump files with MetaFetcheR's bulk insertion functions, and extend the cache on the fly by HTTP-fetching and persisting KEGG and PubChem records.

## When to use

Use when metabolomics identifier mapping requires a local cache across HMDB, ChEBI, LIPID MAPS, KEGG and PubChem, and repeated KEGG/PubChem HTTP queries must be avoided to prevent timeouts from excessive calls.

## When NOT to use

- The identifiers needed are only in databases for which no dump file exists and no on-the-fly fetcher is provided (e.g. METLIN is cited in the paper but no bulk-insert or HTTP caching path is described for it).
- You only need a one-off single-metabolite lookup — building and maintaining a local PostgreSQL cache is overkill compared to direct API queries.
- You need unique MetaFetcheR identifiers: the paper states these are a planned future feature, not currently available.

## Inputs

- HMDB database dump file (downloaded from HMDB)
- ChEBI database dump file
- LIPID MAPS (Lipidomics Gateway) dump file
- Secondary identifier / cross-reference file for bulk_insert_secondary_ids
- MetaFetcheR config settings: host, port, db_name, user, password, path, and dump file names (write_config)
- Sparse primary database identifiers used as seeds for on-the-fly KEGG/PubChem fetching

## Outputs

- Populated local PostgreSQL cache database with tables for HMDB, ChEBI, LIPID MAPS and secondary identifiers
- Cached KEGG records (create_kegg_record) stored in PostgreSQL
- Cached PubChem records (create_pubchem_record) stored in PostgreSQL
- Cross-database identifier mappings across the five supported small-compound databases

## How to apply

Install MetaFetcheR via devtools with install_github("shizidushu/hfun","komorowskilab/metafetcher") and load dependencies (DBI, RPostgreSQL/RPostgres, httr, XML). Write a config file with write_config(...) capturing host, port, db_name, user, password, path and the HMDB/ChEBI/LIPIDMAPS dump file names, then create the empty database with db.create_database(conf) and connect with db.connect(conf), wrapping writes in db.transaction/db.commit. Bulk-load the three downloaded dump files using bulk_insert_hmdb(filepath), bulk_insert_chebi(filepath) and bulk_insert_lipidmaps(filepath), plus bulk_insert_secondary_ids(filepath) for cross-references; format vector columns with convert_df_to_db_array(df, cvectors) before writing tables with db.write_df(table, df). For KEGG and PubChem, which have no dump files, build request URLs with id_to_url(db_id, db_tag), fetch via http_call_api(url, db_id), parse with create_kegg_record / create_pubchem_record and persist the records into the PostgreSQL tables so subsequent queries are served from the cache instead of HTTP. Rationale: the local PostgreSQL database acts as a cache memory of information, avoiding unnecessary HTTP calls and timeouts caused by excessive calls.

## Related tools

- **MetaFetcheR** (R package providing bulk_insert_hmdb/bulk_insert_chebi/bulk_insert_lipidmaps, secondary-ID insertion, config/database management (write_config, db.create_database, db.connect) and on-the-fly KEGG/PubChem caching functions) — https://github.com/komorowskilab/MetaFetcheR/
- **PostgreSQL** (Local database that acts as the cache memory of metabolite information, storing bulk-loaded dump data and cached HTTP-fetched KEGG/PubChem records)
- **R** (Implementation language; package installed via devtools install_github)
- **DBI / RPostgreSQL / RPostgres** (Database connectivity drivers used to open connections and write data frames into PostgreSQL tables)
- **httr** (HTTP layer for issuing on-the-fly KEGG and PubChem requests via http_call_api(url, db_id))

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher"); conf <- write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name); db.create_database(conf); con <- db.connect(conf); bulk_insert_hmdb(hmdb_filepath); bulk_insert_chebi(chebi_filepath); bulk_insert_lipidmaps(lipidmaps_filepath); bulk_insert_secondary_ids(sec_filepath)
```

## Evaluation signals

- Run db.query(SQL) against the PostgreSQL cache and confirm records exist for all five databases (HMDB, ChEBI, LIPID MAPS, KEGG, PubChem).
- Re-query a KEGG or PubChem identifier already in the cache and confirm it is returned from PostgreSQL rather than re-fetched over HTTP (no new network call).
- Vector-valued columns in the dump files round-trip correctly after convert_df_to_db_array(df, cvectors) — array fields are stored as PostgreSQL arrays, not strings.
- bulk_insert_secondary_ids output links primary records to their cross-referenced secondary identifiers across databases.
- The write path runs inside db.transaction/db.commit without errors, and db.write_df(table, df) row counts match the parsed dump file contents.

## Limitations

- HMDB, ChEBI and LIPID MAPS require manually downloaded dump files; only KEGG and PubChem are cached on the fly via HTTP.
- Excessive uncached KEGG/PubChem HTTP calls can cause timeouts, which is precisely why the PostgreSQL cache must be populated before mapping.
- No changelog found and no version number stated for MetaFetcheR in the paper.
- The five databases are not exhaustively re-named in the discussion section; METLIN appears in the intro's database list but no loading path for it is described.
- Unique MetaFetcheR identifiers are only a planned future update, not currently available.

## Evidence
