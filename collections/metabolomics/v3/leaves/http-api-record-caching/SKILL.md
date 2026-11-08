---
name: http-api-record-caching
description: Use when you need KEGG or PubChem metabolite records repeatedly during
  identifier mapping, and remote HTTP fetching must be throttled by a persistent local
  cache to avoid timeouts from excessive calls.
when_to_use_negative:
- When the source database provides dump files — HMDB, ChEBI and LIPID MAPS are loaded
  with bulk_insert_hmdb/bulk_insert_chebi/bulk_insert_lipidmaps, not fetched over
  HTTP
- When the record for a db_id already exists in the PostgreSQL cache — query the cache
  with db.query(SQL) instead of calling the API again
- When no network access or API endpoint is available, since this skill depends on
  live KEGG/PubChem HTTP responses for cache misses
edam_operation: http://edamontology.org/operation_2409
tools:
- name: MetaFetcheR
  role: R package providing the caching functions (id_to_url, http_call_api, create_kegg_record,
    create_pubchem_record) that fetch KEGG/PubChem records on the fly and store them
    in the PostgreSQL cache
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: httr
  role: R dependency used to issue the on-the-fly HTTP calls to KEGG and PubChem APIs
- name: DBI
  role: R database interface used with RPostgreSQL/RPostgres to write fetched records
    into the PostgreSQL cache
- name: RPostgres
  role: PostgreSQL driver alternative for opening the connection and persisting cached
    KEGG/PubChem records
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:c9441d526666035d2a986e6621d94a71d54711437e635267e54e75b00fd62fdc
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
  - R
  - RPostgreSQL
  - RPostgres
  - XML
  - config
  - httr
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# http-api-record-caching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fetches small-compound records from KEGG and PubChem over HTTP only when they are not already present, persisting each parsed record into a local PostgreSQL cache database so repeated queries avoid unnecessary HTTP calls and the timeouts caused by excessive calls.

## When to use

Use when you need KEGG or PubChem metabolite records repeatedly during identifier mapping, and remote HTTP fetching must be throttled by a persistent local cache to avoid timeouts from excessive calls.

## When NOT to use

- When the source database provides dump files — HMDB, ChEBI and LIPID MAPS are loaded with bulk_insert_hmdb/bulk_insert_chebi/bulk_insert_lipidmaps, not fetched over HTTP
- When the record for a db_id already exists in the PostgreSQL cache — query the cache with db.query(SQL) instead of calling the API again
- When no network access or API endpoint is available, since this skill depends on live KEGG/PubChem HTTP responses for cache misses

## Inputs

- KEGG compound identifiers (db_id) with a KEGG database tag
- PubChem compound identifiers (db_id) with a PubChem database tag
- PostgreSQL connection configuration written with write_config(conf) (host, port, db_name, user, password, path)
- Previously cached KEGG/PubChem records stored in the local PostgreSQL database

## Outputs

- Parsed KEGG record objects created by create_kegg_record, persisted as rows in PostgreSQL cache tables
- Parsed PubChem record objects created by create_pubchem_record, persisted as rows in PostgreSQL cache tables
- A populated local PostgreSQL cache that later queries can hit instead of re-issuing HTTP requests

## How to apply

Build a request URL for each identifier with id_to_url(db_id, db_tag), then issue the request with http_call_api(url, db_id) via httr. Before fetching, check the local PostgreSQL database — the underlying algorithm treats this database as a cache memory of information — and only fetch over HTTP when the record is absent. Parse fetched KEGG and PubChem responses with create_kegg_record and create_pubchem_record respectively, then write the parsed record objects into the PostgreSQL tables using DBI/RPostgreSQL or RPostgres, wrapping batched writes in db.transaction/db.commit. Note that only KEGG and PubChem are fetched on the fly: HMDB, ChEBI and LIPID MAPS content enters the cache database via bulk insertion of downloaded dump files, so HTTP caching is deliberately restricted to the two APIs without bulk dumps.

## Related tools

- **MetaFetcheR** (R package providing the caching functions (id_to_url, http_call_api, create_kegg_record, create_pubchem_record) that fetch KEGG/PubChem records on the fly and store them in the PostgreSQL cache) — https://github.com/komorowskilab/MetaFetcheR/
- **httr** (R dependency used to issue the on-the-fly HTTP calls to KEGG and PubChem APIs)
- **DBI** (R database interface used with RPostgreSQL/RPostgres to write fetched records into the PostgreSQL cache)
- **RPostgres** (PostgreSQL driver alternative for opening the connection and persisting cached KEGG/PubChem records)

## Examples

```
url <- id_to_url(db_id, db_tag); rec <- http_call_api(url, db_id)
```

## Evaluation signals

- Run db.query(SQL) against the cache and confirm records exist for all five databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS) after population
- Re-request an already-fetched db_id and verify it is returned from the PostgreSQL cache rather than re-fetched over HTTP
- Confirm that every HTTP cache miss produces a parsed record via create_kegg_record or create_pubchem_record and a corresponding row in the PostgreSQL tables
- Check that repeated bulk mapping runs complete without the timeouts caused by excessive KEGG/PubChem API calls

## Limitations

- Only KEGG and PubChem support on-the-fly HTTP caching; HMDB, ChEBI and LIPID MAPS require pre-downloaded dump files inserted via bulk insertion
- Uniquely identifying records requires a working PostgreSQL instance configured via write_config; the cache is local and must be constructed before HTTP caching can be consulted
- The package covers exactly five small-compound databases; additional databases were only planned as future updates

## Evidence
