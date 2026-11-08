---
name: random-identifier-sampling
description: Use when you need to quantify how reliably MetaFetcheR resolves identifiers
  for a given compound database — e.g. to reproduce the Supplementary Tables S2-S6
  quality test before trusting mapping output.
when_to_use_negative:
- You want to map an actual study's metabolite identifier list — this skill is a randomized
  quality benchmark of the cache database, not a data-fetching workflow
- The local PostgreSQL cache has not been populated via install_databases / bulk_insert_hmdb,
  bulk_insert_chebi, bulk_insert_lipidmaps and on-the-fly KEGG/PubChem caching
- The identifiers of interest belong to databases outside the five supported ones
  (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: MetaFetcheR
  role: Provides do_consistency_test(db, n, attempts) and resolve_single_id(start_db_tag,
    start_db_id) used to sample and repeatedly resolve random identifiers
  repo: github.com/komorowskilab/MetaFetcheR
- name: PostgreSQL
  role: Local cache database whose identifier records are sampled and whose recorded
    test outcomes are queried with db.query(SQL)
- name: R
  role: Language in which MetaFetcheR and the consistency-test functions are implemented
    and invoked
  repo: github.com/komorowskilab/MetaFetcheR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:795da1d89697466636e54d7910bc7dd956d12121a04580b5cc6f7698f6022767
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/komorowskilab/MetaFetcheR
  tools:
  - MetaFetcheR
  - PostgreSQL
  - R
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# random-identifier-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A quality-test procedure for MetaFetcheR in which random identifiers drawn from the five supported small compound databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS) are repeatedly resolved via the queue-based mapping routine to measure per-database consistency, ambiguity, and unresolved rates.

## When to use

Use when you need to quantify how reliably MetaFetcheR resolves identifiers for a given compound database — e.g. to reproduce the Supplementary Tables S2-S6 quality test before trusting mapping output.

## When NOT to use

- You want to map an actual study's metabolite identifier list — this skill is a randomized quality benchmark of the cache database, not a data-fetching workflow
- The local PostgreSQL cache has not been populated via install_databases / bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps and on-the-fly KEGG/PubChem caching
- The identifiers of interest belong to databases outside the five supported ones (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS)

## Inputs

- MetaFetcheR configuration file produced by write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name)
- Populated local PostgreSQL cache database built from HMDB, ChEBI and LIPID MAPS dump files plus on-the-fly cached KEGG and PubChem entries
- Random identifier draws from each of the five supported databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS)

## Outputs

- Per-database result table with percentage of consistent, ambiguous (non-unique), and unresolved resolutions (as in Supplementary Tables S2-S6)
- Single aggregated CSV of the five per-database quality-test result tables

## How to apply

First read the MetaFetcheR configuration file produced by write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name) and open a connection to the populated local PostgreSQL cache with db.connect(conf). For each of the five supported databases, call do_consistency_test(db, n=1000, attempts=100): this samples 1000 random identifiers from that database and, across 100 attempts, resolves each via the queue-based mapping routine resolve_single_id(start_db_tag, start_db_id), which reiterates exhaustively until all identifiers are retrieved or cannot be further resolved. Query the recorded outcomes with db.query(SQL) and compute, per database, the percentage of fully consistent resolutions, ambiguous (non-unique) resolutions, and unresolved cases. Aggregate the five per-database result tables into one CSV and check the rates against the published reference: KEGG should show the highest consistency (~65%) with the lowest unresolved fraction (~23%), while HMDB should show the highest unresolved fraction (~71%).

## Related tools

- **MetaFetcheR** (Provides do_consistency_test(db, n, attempts) and resolve_single_id(start_db_tag, start_db_id) used to sample and repeatedly resolve random identifiers) — github.com/komorowskilab/MetaFetcheR
- **PostgreSQL** (Local cache database whose identifier records are sampled and whose recorded test outcomes are queried with db.query(SQL))
- **R** (Language in which MetaFetcheR and the consistency-test functions are implemented and invoked) — github.com/komorowskilab/MetaFetcheR

## Examples

```
do_consistency_test(db, n=1000, attempts=100)
```

## Evaluation signals

- KEGG achieves the highest consistency (~65%) and the lowest fraction of unresolved cases (~23%) across the 100-times-per-database test
- HMDB exhibits the highest fraction of unresolved cases (~71%)
- Each database result is based on 1000 sampled identifiers evaluated across 100 attempts (do_consistency_test(db, n=1000, attempts=100))
- Percentages of consistent + ambiguous + unresolved cases reconcile per database (rates are complementary outcome categories)
- Aggregated CSV reproduces the pattern reported in Supplementary Tables S2-S6

## Limitations

- Unresolved cases can be substantial for some databases (HMDB ~71%), so results reflect cache completeness as much as algorithm correctness
- Coverage depends on which database dump files (HMDB, ChEBI, LIPID MAPS) were bulk-inserted and which KEGG/PubChem instances were cached on the fly at test time
- The five small compound databases covered are fixed (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS); additional databases were only planned future updates
- No version number of MetaFetcheR is stated, so exact function behavior may vary with the GitHub snapshot installed

## Evidence

- [methods] consistency test invocation: [evidence span withheld — anchor status: not_found]
- [results] reference rates: [evidence span withheld — anchor status: not_found]
