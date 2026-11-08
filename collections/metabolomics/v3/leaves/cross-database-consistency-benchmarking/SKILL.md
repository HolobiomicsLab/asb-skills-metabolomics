---
name: cross-database-consistency-benchmarking
description: Use when you need to quantify how reliably MetaFetcheR maps identifiers
  within one of the five supported small-compound databases (HMDB, ChEBI, PubChem,
  KEGG, LIPID MAPS), by measuring consistency, ambiguity, and unresolved rates over
  random samples.
when_to_use_negative:
- Your identifiers come from a database outside the five supported sources (HMDB,
  ChEBI, PubChem, KEGG, LIPID MAPS) — the sampling test draws identifiers from these
  databases only
- No local PostgreSQL cache database has been populated with the database dump files
  (via install_databases / bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps)
  — the test resolves IDs through the cache, not live fetches alone
- Your goal is one-off mapping of a specific metabolite list rather than measuring
  per-database mapping reliability
tools:
- name: MetaFetcheR
  role: Provides do_consistency_test, resolve_single_id, write_config and db.connect
    used to run the consistency/ambiguity/unresolved sampling test
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: PostgreSQL
  role: Local cache database that stores the bulk-inserted database content queried
    during resolution attempts
- name: R
  role: Language in which MetaFetcheR is implemented and invoked for the test
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:2db553621920b0120128c10b6aa7034172fd7f5fb2423180438526e7632e4454
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-database-consistency-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A stochastic quality test built into MetaFetcheR that repeatedly resolves randomly sampled identifiers from the five supported small-compound databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS) to quantify per-database consistency, ambiguity, and unresolved rates, as reported in Supplementary Tables S2-S6.

## When to use

Use when you need to quantify how reliably MetaFetcheR maps identifiers within one of the five supported small-compound databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS), by measuring consistency, ambiguity, and unresolved rates over random samples.

## When NOT to use

- Your identifiers come from a database outside the five supported sources (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS) — the sampling test draws identifiers from these databases only
- No local PostgreSQL cache database has been populated with the database dump files (via install_databases / bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps) — the test resolves IDs through the cache, not live fetches alone
- Your goal is one-off mapping of a specific metabolite list rather than measuring per-database mapping reliability

## Inputs

- MetaFetcheR configuration file (produced by write_config)
- Populated local PostgreSQL cache database covering HMDB, ChEBI, PubChem, KEGG, and LIPID MAPS
- db connection object from db.connect(conf)
- Randomly sampled identifiers from one of the five supported small-compound databases (n=1000 per database)

## Outputs

- Per-database percentages of consistent, ambiguous (non-unique), and unresolved resolutions
- Single CSV aggregating the five per-database result tables (analogous to Supplementary Tables S2-S6)

## How to apply

First construct and populate the local PostgreSQL cache database (write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name)) and open the connection with db.connect(conf). For each supported database in turn, invoke do_consistency_test(db, n=1000, attempts=100), which samples 1000 random identifiers from that database and repeatedly resolves each via the queue-based mapping routine resolve_single_id(start_db_tag, start_db_id) across 100 attempts. Query the recorded outcomes with db.query(SQL) and compute, per database, the percentage of fully consistent resolutions, ambiguous (non-unique) resolutions, and unresolved cases. Aggregate the five per-database result tables into a single CSV and compare against the published reference values: KEGG ~65% consistency with ~23% unresolved, HMDB ~71% unresolved.

## Related tools

- **MetaFetcheR** (Provides do_consistency_test, resolve_single_id, write_config and db.connect used to run the consistency/ambiguity/unresolved sampling test) — https://github.com/komorowskilab/MetaFetcheR/
- **PostgreSQL** (Local cache database that stores the bulk-inserted database content queried during resolution attempts)
- **R** (Language in which MetaFetcheR is implemented and invoked for the test)

## Examples

```
conf <- write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name); db <- db.connect(conf); results <- do_consistency_test(db, n = 1000, attempts = 100)
```

## Evaluation signals

- KEGG should show the highest consistency rate (~65%) and the lowest fraction of unresolved cases (~23%)
- HMDB should show the highest fraction of unresolved cases (~71%)
- Per-database consistency + ambiguity + unresolved percentages should account for all sampled resolutions (each of the 1000 sampled IDs per database is classified into one of the three outcomes)
- The aggregated CSV reproduces the rates reported in Supplementary Tables S2-S6 when run at n=1000 with 100 attempts
- Every sampled identifier must be resolvable via resolve_single_id(start_db_tag, start_db_id) against the populated PostgreSQL cache; failure indicates an unpopulated or misconfigured cache

## Limitations

- HMDB exhibits a very high fraction of unresolved cases (~71%), so near-complete resolution should not be expected for all databases
- Results are based on random sampling (1000 identifiers, 100 attempts per database), so rates are stochastic estimates rather than exhaustive counts
- The benchmark covers only the five databases supported by MetaFetcheR (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS); METLIN was mentioned in the article but is not in the README's supported list

## Evidence

- [methods] consistency test invocation: [evidence span withheld — anchor status: not_found]
- [methods] queue-based resolution: [evidence span withheld — anchor status: not_found]
- [supplementary] benchmark reference rates: [evidence span withheld — anchor status: not_found]
- [intro] package purpose: "MetaFetcheR is an R package that uses the sparse input of primary database identifiers as a reference point to retrieve identifiers from other databases."
- [readme] supported databases: "Currently the package supports resolving IDs for the following databases"
