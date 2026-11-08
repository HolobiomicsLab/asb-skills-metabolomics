---
name: postgresql-cache-database-querying
description: Use when an identifier-mapping workflow repeatedly resolves metabolite
  identifiers across HMDB, ChEBI, KEGG, PubChem and LIPID MAPS and needs a local cache
  to avoid redundant HTTP fetches.
when_to_use_negative:
- The task only needs a single one-off identifier lookup and no repeated resolution
  — direct API calls to KEGG/PubChem may suffice without building the cache
- The metabolites of interest lie outside the five supported small compound databases
  (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS), e.g. METLIN-only identifiers
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: MetaFetcheR
  role: Provides db.connect, write_config, bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps,
    bulk_insert_secondary_ids, db.query, db.write_df and do_consistency_test used
    to build, query and verify the PostgreSQL cache
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: PostgreSQL
  role: Local database engine acting as a cache memory of identifier information read
    and written by the queue-based algorithm
- name: R
  role: Implementation language in which the cache-population and querying functions
    are invoked
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_002
  - task_004
  - task_001
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:e4b720f36617549a76a511775a788f5cbcb52e494387ec64b40e2971819150ea
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
  - MetaFetcheR
  - PostgreSQL
  - R
  - chebi
  - hmdb
  - kegg
  - lipidmaps
  - pubchem
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# postgresql-cache-database-querying

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct and query a local PostgreSQL cache database that underlies MetaFetcheR, storing HMDB, ChEBI and LIPID MAPS records from downloaded dump files via bulk insertion and accumulating KEGG and PubChem instances cached on the fly through HTTP calls.

## When to use

Use when an identifier-mapping workflow repeatedly resolves metabolite identifiers across HMDB, ChEBI, KEGG, PubChem and LIPID MAPS and needs a local cache to avoid redundant HTTP fetches.

## When NOT to use

- The task only needs a single one-off identifier lookup and no repeated resolution — direct API calls to KEGG/PubChem may suffice without building the cache
- The metabolites of interest lie outside the five supported small compound databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS), e.g. METLIN-only identifiers

## Inputs

- HMDB, ChEBI and LIPID MAPS database dump files (paths supplied to write_config)
- MetaFetcheR configuration object (host, port, db_name, user, password, path)
- Discovered metabolite identifier data frames (e.g. df.discovered) whose resolutions are written back to the cache

## Outputs

- Populated local PostgreSQL cache database containing HMDB, ChEBI, LIPID MAPS and on-the-fly cached KEGG and PubChem records
- Query result tables (e.g. per-database consistency/ambiguity/unresolved rates from db.query(SQL))
- CSV exports such as resolved_metabolite_mappings.csv and consistency-test result tables (Supplementary Tables S2-S6)

## How to apply

Build the configuration with write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name) after downloading the database dump files, then populate the cache by invoking the bulk insertion functions bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps and bulk_insert_secondary_ids. Open the connection with db.connect(conf) so that the queue-based algorithm can read cached records and write newly discovered instances. Since KEGG and PubChem lack dump files, their records are fetched on the fly via http_call_api(url, db_id) with URLs built by id_to_url(db_id, db_tag) and then persisted to PostgreSQL inside a db.transaction / db.commit cycle using db.write_df. Read cached values with db.query(SQL), flag empties with null2na(v) and is.empty(v), and store ambiguous multi-mapped identifiers as joined arrays via join_sql_arr(v). Cache formerly discovered records so subsequent runs skip redundant iterations, and verify cache integrity with do_consistency_test(db, n, attempts, offset).

## Related tools

- **MetaFetcheR** (Provides db.connect, write_config, bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps, bulk_insert_secondary_ids, db.query, db.write_df and do_consistency_test used to build, query and verify the PostgreSQL cache) — https://github.com/komorowskilab/MetaFetcheR/
- **PostgreSQL** (Local database engine acting as a cache memory of identifier information read and written by the queue-based algorithm)
- **R** (Implementation language in which the cache-population and querying functions are invoked)

## Examples

```
conf <- write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name); db <- db.connect(conf); do_consistency_test(db, n=1000, attempts=100)
```

## Evaluation signals

- do_consistency_test(db, n=1000, attempts=100) per database resolves sampled identifiers matching cached database contents without unresolved empty fields for resolvable records
- Reported per-database rates reproduce Supplementary Tables S2-S6 patterns: KEGG highest consistency (~65%) with lowest unresolved (~23%), HMDB highest unresolved (~71%)
- Every newly cached record is committed within a db.transaction / db.commit cycle and ambiguous multi-mappings are stored as joined arrays via join_sql_arr
- Subsequent runs of resolve_metabolites skip redundant iterations because formerly discovered records were cached
- Empty identifier fields are consistently detected with null2na(v) and is.empty(v) before any write-back

## Limitations

- Only HMDB, ChEBI and LIPID MAPS are populated from dump files; KEGG and PubChem depend on on-the-fly HTTP calls, so coverage grows incrementally with use
- In the 100-times-per-database quality test, HMDB showed the highest fraction of unresolved cases (~71%), so a large share of HMDB-seeded resolutions may not complete
- No changelog was found and no version number is stated for MetaFetcheR; unique MetaFetcheR identifiers are only a planned future feature
- Benchmark datasets are cited only by publication reference (Diamanti et al., 2019; Priolo et al., 2014) with no accession or data availability information

## Evidence

- [supplementary] KEGG showed the highest consistency (~65%) and the lowest fraction of unresolved cases (~23%), while HMDB had the highest fraction of unresolved cases (~71%).: [evidence span withheld — anchor status: not_found]
