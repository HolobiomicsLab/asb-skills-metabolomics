---
name: queue-based-graph-traversal-identifier-filling
description: Use when a metabolite table (e.g. HMDB IDs) has sparse/empty identifier
  fields across databases (KEGG, PubChem, ChEBI, LIPID MAPS, METLIN) that must be
  resolved for pathway enrichment analysis.
when_to_use_negative:
- The metabolite names/identifiers are already fully mapped and no empty foreign-reference
  fields remain to be filled
- The task needs a unique MetaFetcheR identifier — the paper states this is only a
  planned future feature, not currently available
- A quick single-database mapping to HMDB suffices — the article notes mapping to
  a single database (HMDB) can cause loss of information, but if downstream tooling
  requires only HMDB IDs, the queue-based traversal is unnecessary overhead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaFetcheR
  role: R package implementing the queue-based resolver (resolve_metabolites, resolve_single_id,
    id_to_url, http_call_api) over five small-compound databases
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: PostgreSQL
  role: Local cache database storing database dump records and on-the-fly fetched
    KEGG/PubChem instances for the traversal to fill empty fields from
- name: R
  role: Implementation language; installed via devtools install_github
  repo: https://github.com/komorowskilab/MetaFetcheR/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:d2ed9ca2f9c5f036d92297ea899b85b8a3bda64edf1d3fc837a351239bb13aac
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# queue-based-graph-traversal-identifier-filling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MetaFetcheR's queue-based algorithm iteratively fills empty metabolite identifier fields by traversing cross-database mappings cached in a local PostgreSQL database, reiterating exhaustively until all identifiers are retrieved or can no longer be resolved. It handles ambiguous multi-mapped compounds by flagging them for user selection rather than forcing a single mapping.

## When to use

Use when a metabolite table (e.g. HMDB IDs) has sparse/empty identifier fields across databases (KEGG, PubChem, ChEBI, LIPID MAPS, METLIN) that must be resolved for pathway enrichment analysis.

## When NOT to use

- The metabolite names/identifiers are already fully mapped and no empty foreign-reference fields remain to be filled
- The task needs a unique MetaFetcheR identifier — the paper states this is only a planned future feature, not currently available
- A quick single-database mapping to HMDB suffices — the article notes mapping to a single database (HMDB) can cause loss of information, but if downstream tooling requires only HMDB IDs, the queue-based traversal is unnecessary overhead

## Inputs

- Reference metabolite table (df.discovered) with known primary database identifiers (e.g. HMDB IDs) and empty fields for other databases
- Local PostgreSQL cache database populated from HMDB, ChEBI and LIPID MAPS dump files via bulk insertion (bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps, bulk_insert_secondary_ids)
- Configuration object for db.connect(conf)

## Outputs

- Cross-database identifier mapping table written to resolved_metabolite_mappings.csv
- Ambiguous multi-mapped entries flagged for user selection of a representative identifier
- PostgreSQL cache updated with on-the-fly fetched KEGG and PubChem records

## How to apply

Load the reference metabolite table (df.discovered) containing known primary database identifiers and connect to the local PostgreSQL cache with db.connect(conf). Run resolve_metabolites(df.discovered), which enqueues records and iteratively fills empty identifier fields using resolve_single_id(start_db_tag, start_db_id) as the per-record entry point; the queue is revisited so that identifiers resolved for one compound can seed further lookups. For identifiers missing from the cache, call the KEGG and PubChem REST APIs on the fly via http_call_api(url, db_id) with URLs built by id_to_url(db_id, db_tag), and write newly cached instances back into PostgreSQL. Mark entries mapping to multiple candidate compounds as ambiguous for user selection instead of forcing a single mapping. Reiterate until all identifiers are retrieved or cannot be further resolved, then export the cross-database mapping table.

## Related tools

- **MetaFetcheR** (R package implementing the queue-based resolver (resolve_metabolites, resolve_single_id, id_to_url, http_call_api) over five small-compound databases) — https://github.com/komorowskilab/MetaFetcheR/
- **PostgreSQL** (Local cache database storing database dump records and on-the-fly fetched KEGG/PubChem instances for the traversal to fill empty fields from)
- **R** (Implementation language; installed via devtools install_github) — https://github.com/komorowskilab/MetaFetcheR/

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher")
```

## Evaluation signals

- All empty identifier fields in the output table are either filled or explicitly marked unresolvable — the loop terminates only when 'all identifiers have been retrieved or cannot be further resolved'
- Ambiguous records (one identifier mapping to multiple candidate compounds) are flagged for user selection, not silently collapsed to one mapping
- resolved_metabolite_mappings.csv is produced and its HMDB-to-KEGG mappings match the reference mapping generated by MetaFetcheR for the metabolite set
- Newly fetched KEGG/PubChem records are persisted in the PostgreSQL cache so re-runs resolve from cache without repeated HTTP calls
- Resolved mappings outperform MS_targeted and MetaboAnalyst mappings on the same datasets (Diamanti et al., 2019; Priolo et al., 2014) in coverage

## Limitations

- Exceptional cases of empty returns from a query require exhaustive reiteration, so resolution may terminate with some identifiers unretrievable
- Only five databases are covered (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS); coverage of others depends on continuous future updates
- Unique MetaFetcheR identifiers do not exist yet — they are only a planned future update
- The benchmark comparison against MS_targeted and MetaboAnalyst reports no quantitative performance metrics
- HMDB, ChEBI and LIPID MAPS require pre-downloaded dump files and bulk insertion before traversal; KEGG and PubChem depend on live HTTP calls, so network availability affects resolution

## Evidence
