---
name: mapping-rate-aggregation-statistics
description: Use when you need to measure how consistently MetaFetcheR resolves metabolite
  identifiers across the five supported small-compound databases, e.g. to validate
  a populated PostgreSQL cache or compare mapping quality per source database.
when_to_use_negative:
- The PostgreSQL cache is empty or incompletely populated (bulk_insert_hmdb, bulk_insert_chebi,
  bulk_insert_lipidmaps, bulk_insert_secondary_ids have not been run) — sampling random
  identifiers from an unpopulated database yields meaningless rates
- The goal is mapping metabolite identifiers for a downstream analysis (e.g. pathway
  enrichment) rather than benchmarking mapping quality — use the standard MetaFetcheR
  mapping workflow instead
- The identifier source is user-supplied experimental data rather than randomly sampled
  identifiers from the five supported databases — the test's rates only characterize
  the databases themselves
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: MetaFetcheR
  role: Provides do_consistency_test(db, n=1000, attempts=100), resolve_single_id(start_db_tag,
    start_db_id), db.connect(conf) and db.query(SQL) used to run the sampling-based
    consistency test over the PostgreSQL cache
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: R
  role: Implementation language in which the consistency test is invoked and per-database
    rates are computed and aggregated into CSV
- name: PostgreSQL
  role: Local cache database holding the mapping data that the queue-based resolution
    routine queries during each of the 100 attempts per identifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:dc33b0f59fd98bb120ea4a8028d3226b16d3af45df65ba97f4f05f6b08f6c9c6
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
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mapping-rate-aggregation-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the reliability of MetaFetcheR's cross-database identifier mapping by repeatedly resolving randomly sampled identifiers from HMDB, ChEBI, PubChem, KEGG and LIPID MAPS and computing per-database consistency, ambiguity, and unresolved rates. It supplies the benchmark numbers (e.g. KEGG ~65% consistency, HMDB ~71% unresolved) reported in Supplementary Tables S2-S6.

## When to use

Use when you need to measure how consistently MetaFetcheR resolves metabolite identifiers across the five supported small-compound databases, e.g. to validate a populated PostgreSQL cache or compare mapping quality per source database.

## When NOT to use

- The PostgreSQL cache is empty or incompletely populated (bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps, bulk_insert_secondary_ids have not been run) — sampling random identifiers from an unpopulated database yields meaningless rates
- The goal is mapping metabolite identifiers for a downstream analysis (e.g. pathway enrichment) rather than benchmarking mapping quality — use the standard MetaFetcheR mapping workflow instead
- The identifier source is user-supplied experimental data rather than randomly sampled identifiers from the five supported databases — the test's rates only characterize the databases themselves

## Inputs

- MetaFetcheR configuration file produced by write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name)
- Populated local PostgreSQL cache database containing bulk-inserted HMDB, ChEBI and LIPID MAPS dumps and on-the-fly cached KEGG and PubChem instances
- SQL query string passed to db.query(SQL) to retrieve recorded resolution outcomes

## Outputs

- Per-database percentages of consistent, ambiguous (non-unique), and unresolved resolutions (1000 sampled identifiers, 100 attempts each)
- Single aggregated CSV compiling the five per-database result tables, comparable to Supplementary Tables S2-S6

## How to apply

First read the MetaFetcheR configuration file produced by write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name) and open a connection to the populated local PostgreSQL cache with db.connect(conf). Then, for each of the five supported databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS), invoke do_consistency_test(db, n=1000, attempts=100), which samples 1000 random identifiers from that database and repeatedly resolves each one through the queue-based mapping routine resolve_single_id(start_db_tag, start_db_id) across 100 attempts — the queue-based algorithm reiterates exhaustively until all identifiers are retrieved or cannot be further resolved. Query the recorded outcomes with db.query(SQL) and compute per database the percentage of fully consistent resolutions, ambiguous (non-unique) resolutions, and unresolved cases. Finally aggregate the five per-database result tables into a single CSV and compare the compiled rates against the published Supplementary Tables S2-S6 as a sanity check.

## Related tools

- **MetaFetcheR** (Provides do_consistency_test(db, n=1000, attempts=100), resolve_single_id(start_db_tag, start_db_id), db.connect(conf) and db.query(SQL) used to run the sampling-based consistency test over the PostgreSQL cache) — https://github.com/komorowskilab/MetaFetcheR/
- **R** (Implementation language in which the consistency test is invoked and per-database rates are computed and aggregated into CSV)
- **PostgreSQL** (Local cache database holding the mapping data that the queue-based resolution routine queries during each of the 100 attempts per identifier)

## Examples

```
conf <- write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name); db <- db.connect(conf); res <- do_consistency_test(db, n=1000, attempts=100)
```

## Evaluation signals

- KEGG should show the highest consistency (~65%) and the lowest fraction of unresolved cases (~23%)
- HMDB should show the highest fraction of unresolved cases (~71%)
- Exactly five per-database result tables are produced (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS), each based on n=1000 sampled identifiers resolved over 100 attempts
- Per database, the three outcome classes (consistent, ambiguous, unresolved) should account for essentially all sampled identifiers
- Aggregated CSV rates should reproduce the published Supplementary Tables S2-S6 within sampling error

## Limitations

- Rates are computed by sampling 1000 random identifiers per database with 100 resolution attempts, so estimates carry sampling variability and are not exhaustive over all identifiers
- A substantial fraction of identifiers remains unresolved even with the exhaustive queue-based algorithm (e.g. ~23% for KEGG, ~71% for HMDB), reflecting missing foreign reference identifiers across databases
- Unique MetaFetcheR identifiers were only a planned future feature at publication time, so ambiguous (non-unique) resolutions cannot be collapsed into a single canonical ID
- The benchmark comparison against MS_targeted and MetaboAnalyst lacks quantitative performance metrics in the discussion

## Evidence
