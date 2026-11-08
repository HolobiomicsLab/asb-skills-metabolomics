---
name: queue-based-resolution-algorithm-implementation
description: Use when a metabolomics dataset contains sparse primary identifiers (e.g.
  HMDB, KEGG, PubChem) and you must map them to identifiers of other small-compound
  databases, resolving empty/ambiguous returns by exhaustive re-iteration.
when_to_use_negative:
- Your metabolites already carry complete, unambiguous identifier sets for all target
  databases, so no empty fields need resolving
- You only need mapping to a single database (e.g. HMDB) via existing tools like MS_targeted
  or MetaboAnalyst rather than exhaustive cross-database resolution
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: MetaFetcheR
  role: R package implementing the queue-based resolution algorithm, PostgreSQL caching
    and consistency test
  repo: https://github.com/komorowskilab/MetaFetcheR
- name: PostgreSQL
  role: local cache database read/written by the queue-based algorithm
- name: KEGG
  role: on-the-fly identifier source queried via HTTP calls
- name: PubChem
  role: on-the-fly identifier source queried via HTTP calls
- name: HMDB
  role: bulk-inserted dump file used to seed the cache
- name: ChEBI
  role: bulk-inserted dump file used to seed the cache
- name: LIPID MAPS
  role: bulk-inserted dump file used to seed the cache
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:519b485758e696880fdc92c1690564b0b7de6724e91406b618ebba7454e6d173
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
  - ChEBI
  - HMDB
  - KEGG
  - LIPID MAPS
  - MetaFetcheR
  - PostgreSQL
  - PubChem
  - R
  - chebi
  - hmdb
  - kegg
  - lipidmaps
  - pubchem
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# queue-based-resolution-algorithm-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implement MetaFetcheR's queue-based identifier-resolution algorithm: iteratively fill empty foreign-reference-identifier fields for each metabolite from a PostgreSQL cache (HMDB, ChEBI, LIPID MAPS) and on-the-fly HTTP calls (KEGG, PubChem), reiterating until all retrievable identifiers are exhausted.

## When to use

Use when a metabolomics dataset contains sparse primary identifiers (e.g. HMDB, KEGG, PubChem) and you must map them to identifiers of other small-compound databases, resolving empty/ambiguous returns by exhaustive re-iteration.

## When NOT to use

- Your metabolites already carry complete, unambiguous identifier sets for all target databases, so no empty fields need resolving
- You only need mapping to a single database (e.g. HMDB) via existing tools like MS_targeted or MetaboAnalyst rather than exhaustive cross-database resolution

## Inputs

- Sparse metabolite identifier table with primary identifiers from HMDB, ChEBI, KEGG, PubChem or LIPID MAPS
- Database dump files for HMDB, ChEBI and LIPID MAPS to populate the local PostgreSQL cache
- HTTP-call access to KEGG and PubChem APIs for on-the-fly caching

## Outputs

- Identifier mapping table with empty fields filled across HMDB, ChEBI, KEGG, PubChem and LIPID MAPS
- PostgreSQL cache database holding both bulk-inserted and newly discovered records
- Consistency-test report for a sample of n cached records

## How to apply

Construct a local PostgreSQL cache database from the dump files of HMDB, ChEBI and LIPID MAPS via MetaFetcheR's bulk insertion functions (bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps), then use queue-based iteration: for each compound, call the resolution function with its known identifiers as a starting point and fill empty identifier fields by reading cached records and issuing HTTP calls to KEGG and PubChem on the fly. When a query returns an empty result, re-enqueue the record and reiterate exhaustively until all identifiers have been retrieved or cannot be further resolved, rather than dropping the record. Cache every newly discovered record back into the PostgreSQL database (wrapped in a transaction with db.write_df / db.commit) so subsequent runs skip redundant queries, and store multiple ambiguous mapped identifiers as joined arrays. Verify correctness with do_consistency_test(db, n, attempts, offset) on a sample of cached records; the algorithm's rationale is that cross-database chains (e.g. HMDB→PubChem→KEGG) let previously unmapped compounds gain identifiers in later iterations.

## Related tools

- **MetaFetcheR** (R package implementing the queue-based resolution algorithm, PostgreSQL caching and consistency test) — https://github.com/komorowskilab/MetaFetcheR
- **PostgreSQL** (local cache database read/written by the queue-based algorithm)
- **KEGG** (on-the-fly identifier source queried via HTTP calls)
- **PubChem** (on-the-fly identifier source queried via HTTP calls)
- **HMDB** (bulk-inserted dump file used to seed the cache)
- **ChEBI** (bulk-inserted dump file used to seed the cache)
- **LIPID MAPS** (bulk-inserted dump file used to seed the cache)

## Examples

```
install_github("shizidushu/hfun","komorowskilab/metafetcher")
```

## Evaluation signals

- Run do_consistency_test(db, n, attempts, offset) on a sample of records and confirm the resolved mappings match cached database contents without unresolved empty fields for records that can be further resolved
- Records that cannot be further resolved are the only ones left with empty identifier fields — an invariant of the exhaustive re-iteration loop
- Multiple ambiguous mapped identifiers for one compound are stored as joined arrays in the PostgreSQL tables, so re-running the algorithm should not change row counts for already-cached compounds
- Newly discovered KEGG/PubChem records must be persisted after each iteration, so a second run skips redundant HTTP calls
- Benchmark comparison against MS_targeted and MetaboAnalyst on published datasets (Diamanti et al., 2019; Priolo et al., 2014) should show MetaFetcheR resolving more identifiers

## Limitations

- The algorithm cannot guarantee resolution when no cross-database chain exists; identifiers remain empty after exhaustive re-iteration
- Empty returns trigger re-iteration, so runtime depends on how many compounds are initially unresolved; no cutoff is reported in the article
- MetaFetcheR currently covers only HMDB, ChEBI, PubChem, KEGG and LIPID MAPS; additional databases are planned but not yet implemented
- Unique MetaFetcheR identifiers are a future feature, not currently available

## Evidence
