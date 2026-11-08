---
name: cross-database-identifier-resolution
description: Use when a sparse metabolite table has known primary identifiers (e.g.
  HMDB IDs) but empty fields for other databases, and you need complete cross-database
  mapping for pathway enrichment or annotation.
when_to_use_negative:
- 'You need stable unique cross-database identifiers: unique MetaFetcheR identifiers
  are only a planned future feature, not currently available'
- 'You need a fully automated one-shot mapper: ambiguous multi-mapped entries require
  the user to choose the representative identifier among the discovered ones'
- You cannot set up a local PostgreSQL database or lack access to HMDB/ChEBI/LIPID
  MAPS dump files, since the cache is the foundation of the algorithm
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: MetaFetcheR
  role: Core R package whose queue-based resolver fills empty identifier fields across
    the five databases; provides resolve_metabolites, bulk_insert_* and http_call_api
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: R
  role: Implementation language; skill is applied as an R workflow using devtools::install_github
- name: PostgreSQL
  role: Local cache database storing information from database dump files and on-the-fly
    cached KEGG/PubChem instances
- name: MetaboAnalyst
  role: Existing mapping tool used as benchmark comparator for the non-empty-field
    mapping rate (~48% and ~62% on the two case-study datasets)
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:ddd6d47758257fe0a69b0483cf96a9371ddca7aab13d21964a9e4d6d20bd068f
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
  - MetaboAnalyst
  - PostgreSQL
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-database-identifier-resolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resolve metabolite identifiers across HMDB, ChEBI, PubChem, KEGG and LIPID MAPS using MetaFetcheR's queue-based algorithm, backed by a local PostgreSQL cache populated from database dump files and on-the-fly HTTP calls. The skill targets filling empty (missing) identifier fields exhaustively so metabolite records can be linked across five small-compound databases.

## When to use

Use when a sparse metabolite table has known primary identifiers (e.g. HMDB IDs) but empty fields for other databases, and you need complete cross-database mapping for pathway enrichment or annotation.

## When NOT to use

- You need stable unique cross-database identifiers: unique MetaFetcheR identifiers are only a planned future feature, not currently available
- You need a fully automated one-shot mapper: ambiguous multi-mapped entries require the user to choose the representative identifier among the discovered ones
- You cannot set up a local PostgreSQL database or lack access to HMDB/ChEBI/LIPID MAPS dump files, since the cache is the foundation of the algorithm

## Inputs

- Sparse metabolite identifier table (df.discovered) with known primary database identifiers and empty fields for other databases
- HMDB database dump file (HMDB_file_name)
- ChEBI database dump file (ChEBI_file_name)
- LIPID MAPS database dump file (LIPIDMAPS_file_name)
- PostgreSQL connection configuration (host, port, db_name, user, password, path)

## Outputs

- Resolved cross-database mapping table (e.g. resolved_metabolite_mappings.csv) with identifiers across HMDB, ChEBI, PubChem, KEGG and LIPID MAPS
- Non-empty-field mapping rate (e.g. mapping_rate_diamanti.csv, mapping_rate_priolo2014.csv) with computed and reference rates
- Populated local PostgreSQL cache including on-the-fly cached KEGG and PubChem records
- User-selected representative identifiers for ambiguous multi-mapped entries

## How to apply

Install MetaFetcheR with install_github('shizidushu/hfun','komorowskilab/metafetcher'), then build a local PostgreSQL cache by downloading HMDB, ChEBI and LIPID MAPS dump files and running bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps and bulk_insert_secondary_ids; connect with db.connect(conf) after configuring via write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name). KEGG and PubChem have no dump files, so their records are cached on the fly with http_call_api(url, db_id) using URLs built by id_to_url(db_id, db_tag), and new instances are stored back in PostgreSQL. Load the sparse identifier table as df.discovered and run resolve_metabolites(df.discovered), whose queue-based algorithm starts from known primary identifiers via resolve_single_id(start_db_tag, start_db_id) and reiterates exhaustively until all identifiers are retrieved or cannot be further resolved. Entries mapping to multiple candidate compounds are marked ambiguous for user selection of a representative identifier rather than forcing a single mapping. Compute the non-empty-field mapping rate with the null2na and is.empty helpers as the fraction of resolved identifier fields over input entries (benchmarks: ~80% vs ~48% MetaboAnalyst on Diamanti et al. 2019; ~78% vs ~62% on Priolo et al. 2014).

## Related tools

- **MetaFetcheR** (Core R package whose queue-based resolver fills empty identifier fields across the five databases; provides resolve_metabolites, bulk_insert_* and http_call_api) — https://github.com/komorowskilab/MetaFetcheR/
- **R** (Implementation language; skill is applied as an R workflow using devtools::install_github)
- **PostgreSQL** (Local cache database storing information from database dump files and on-the-fly cached KEGG/PubChem instances)
- **MetaboAnalyst** (Existing mapping tool used as benchmark comparator for the non-empty-field mapping rate (~48% and ~62% on the two case-study datasets))

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher"); resolved <- resolve_metabolites(df.discovered)
```

## Evaluation signals

- Script runs to completion and the output CSV contains both the computed mapping rate and the reported reference rates
- Computed non-empty-field mapping rate is consistent with the paper's reported values (~80% on Diamanti et al. 2019, ~78% on Priolo et al. 2014, reviewed as closer than MetaboAnalyst's ~48% and ~62%)
- Resolved mapping table contains non-empty fields in multiple target databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS), checked with is.empty and null2na helpers
- PostgreSQL cache is populated: bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps and bulk_insert_secondary_ids complete, and on-the-fly KEGG/PubChem records are stored back
- The resolution loop terminates: identifiers are either retrieved or explicitly marked as unable to be further resolved

## Limitations

- Unique MetaFetcheR identifiers are only a planned future update, not currently available
- No exact acceptance threshold is stated for reproducing the benchmark rates; consistency with ~80%/~78% relies on expert review of closeness
- Ambiguous cases (multiple identifiers for the same compound from the same database) require manual user selection of a representative identifier
- Only HMDB, ChEBI, PubChem, KEGG and LIPID MAPS are supported; additional databases are a planned continuous update

## Evidence

- [intro] Queue-based algorithm resolves identifiers via exhaustive reiteration: "This is orchestrated via a queue-based algorithm. The algorithm handles exceptional cases of empty returns from a query by reiterating exhaustively until all identifiers have been retrieved or cannot"
- [intro] PostgreSQL cache is the foundation of the algorithm: "The foundation of the underlying algorithm relies on constructing a local PostgreSQL database that acts as a cache memory of information."
- [other] Ambiguous multi-mappings are resolved by user choice: [evidence span withheld — anchor status: not_found]
