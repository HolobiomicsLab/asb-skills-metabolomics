---
name: ambiguous-identifier-aggregation
description: Use when a metabolomics table has sparse or missing cross-database identifiers
  (HMDB, ChEBI, KEGG, PubChem, LIPID MAPS) and records of the same metabolite must
  be linked across databases despite mismatches and missing foreign references.
when_to_use_negative:
- The compound database of interest is outside the five supported databases — the
  package supports only HMDB, ChEBI, PubChem, KEGG and LIPID MAPS (METLIN is cited
  in the paper but is not in the supported list)
- Input records already carry complete, unambiguous identifiers in every target database,
  so no empty-field resolution is needed
- You cannot set up a local PostgreSQL database and download the HMDB/ChEBI/LIPID
  MAPS dump files, since the cache-based algorithm depends on them
edam_operation: http://edamontology.org/operation_0335
edam_topics: []
tools:
- name: MetaFetcheR
  role: Core R package implementing the queue-based identifier-resolution algorithm
    with exhaustive re-iteration
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: PostgreSQL
  role: Local cache database storing database dump records and cached KEGG/PubChem
    HTTP results; read and written by the queue-based algorithm
- name: hfun
  role: Helper package installed alongside metafetcher, required for the installation
    to work
  repo: https://github.com/shizidushu/hfun
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:669c655b825526da123627b516a1753cc9f3fc1dd6b61795643a08f1f901e1f8
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
  - hfun
  - hmdb
  - kegg
  - lipidmaps
  - pubchem
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# ambiguous-identifier-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A queue-based identifier-resolution method, implemented in the MetaFetcheR R package, that maps sparse primary metabolite identifiers (HMDB, ChEBI, KEGG, PubChem, LIPID MAPS) to identifiers in other databases by filling empty fields and reiterating exhaustively until no further resolution is possible. It stores multiple ambiguous mapped identifiers per compound to avoid the information loss of single-database (e.g. HMDB-only) mapping.

## When to use

Use when a metabolomics table has sparse or missing cross-database identifiers (HMDB, ChEBI, KEGG, PubChem, LIPID MAPS) and records of the same metabolite must be linked across databases despite mismatches and missing foreign references.

## When NOT to use

- The compound database of interest is outside the five supported databases — the package supports only HMDB, ChEBI, PubChem, KEGG and LIPID MAPS (METLIN is cited in the paper but is not in the supported list)
- Input records already carry complete, unambiguous identifiers in every target database, so no empty-field resolution is needed
- You cannot set up a local PostgreSQL database and download the HMDB/ChEBI/LIPID MAPS dump files, since the cache-based algorithm depends on them

## Inputs

- Database dump files from HMDB, ChEBI and LIPID MAPS (for bulk insertion into the local PostgreSQL cache)
- A metabolite table containing sparse primary database identifiers (e.g. HMDB accession IDs)
- PostgreSQL cache database configuration (conf) used with db.connect(conf)
- Validation parameters: sample size n, number of attempts, and offset for do_consistency_test

## Outputs

- A populated PostgreSQL cache with cross-database identifier mappings per metabolite across HMDB, ChEBI, KEGG, PubChem and LIPID MAPS
- Per-compound identifier sets with multiple ambiguous mapped identifiers stored as joined arrays (join_sql_arr format)
- Consistency-test results confirming resolved mappings match cached database contents
- A mapping table usable for pathway enrichment analysis requiring predefined unique identifier sets

## How to apply

Build a local PostgreSQL cache by downloading HMDB, ChEBI and LIPID MAPS dump files and running the bulk-insertion functions bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps (plus bulk_insert_secondary_ids for secondary identifiers). Install the package via devtools::install_github("shizidushu/hfun","komorowskilab/metafetcher") since it depends on the hfun helper package. For each compound, query the cached PostgreSQL tables for HMDB/ChEBI/LIPID MAPS identifiers and fetch KEGG and PubChem identifiers on the fly via http_call_api(url, db_id) using id_to_url(db_id, db_tag), writing results back inside a db.transaction / db.commit cycle with db.write_df. When a query returns an empty field, do not stop: push the discovered records through resolve_metabolites(df.discovered) and resolve_single_id(start_db_tag, start_db_id) in an exhaustive queue-based re-iteration until every identifier is retrieved or provably unresolvable. Store multiple ambiguous mapped identifiers per compound as joined arrays with join_sql_arr(v) rather than picking a single match, and validate a sample of n records with do_consistency_test(db, n, attempts, offset).

## Related tools

- **MetaFetcheR** (Core R package implementing the queue-based identifier-resolution algorithm with exhaustive re-iteration) — https://github.com/komorowskilab/MetaFetcheR/
- **PostgreSQL** (Local cache database storing database dump records and cached KEGG/PubChem HTTP results; read and written by the queue-based algorithm)
- **hfun** (Helper package installed alongside metafetcher, required for the installation to work) — https://github.com/shizidushu/hfun

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher")
```

## Evaluation signals

- do_consistency_test(db, n, attempts, offset) on a sample of n records returns resolved identifier mappings that match the cached PostgreSQL contents
- No unresolved empty identifier fields remain for records that can be further resolved (checked via null2na(v) / is.empty(v))
- Records with multiple ambiguous mappings are stored as joined arrays via join_sql_arr(v) rather than a single chosen identifier
- Re-running the pipeline skips redundant iterations because formerly discovered records are cached in PostgreSQL
- Case-study benchmark against MS_targeted and MetaboAnalyst shows MetaFetcheR recovers more cross-database links

## Limitations

- The benchmark against MS_targeted and MetaboAnalyst lacks reported quantitative performance metrics
- Unique MetaFetcheR identifiers are only a planned future feature, not currently available
- Requires local PostgreSQL setup and prior download of HMDB, ChEBI and LIPID MAPS dump files; KEGG and PubChem data are only cached on the fly via HTTP calls, so network availability affects resolution
- The five small compound databases covered are not explicitly enumerated in the discussion, and no version number is stated for the package

## Evidence

- [intro] Queue-based algorithm maps known identifiers to identifiers of other databases by filling empty fields, reiterating until all identifiers are retrieved or cannot be further resolved: "This is orchestrated via a queue-based algorithm. The algorithm handles exceptional cases of empty returns from a query by reiterating exhaustively until all identifiers have been retrieved or cannot"
- [intro] Cache data on the fly via HTTP calls from KEGG and PubChem and store cached instances in the PostgreSQL database: "we chose MetaFetcheR to cache data through HTTP calls on the fly from KEGG and PubChem. Cached instances from KEGG and PubChem are stored in the PostgreSQL database"
- [intro] Construct a local PostgreSQL cache database from database dump files via bulk insertion: "database dump files provided by HMDB, ChEBI and LIPID MAPS are to be downloaded. Subsequently, the bulk insertion function of MetaFetcheR is invoked to construct the local database"
- [readme] Installation command for the package and its hfun dependency: "install_github("shizidushu/hfun","komorowskilab/metafetcher")"
- [readme] Package purpose: linking metabolite IDs across metabolome databases to resolve ambiguity: "An R package designed to link metabolites IDs from different Metabolome databases with eachother in a step to resolve ambiguity and standardize metabolites representation and annotation."
