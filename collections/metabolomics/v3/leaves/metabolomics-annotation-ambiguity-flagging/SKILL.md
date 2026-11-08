---
name: metabolomics-annotation-ambiguity-flagging
description: Use when resolving metabolite identifiers across databases and an identifier
  maps to multiple candidate compounds from the same database, so ambiguity must be
  surfaced for user selection instead of silently forcing one mapping.
when_to_use_negative:
- The entity space is not small compounds (e.g., gene or protein identifiers) — MetaFetcheR
  covers only HMDB, ChEBI, PubChem, KEGG and LIPID MAPS
- Every metabolite already has a complete, unambiguous cross-database identifier set
  with no multi-mapped candidates
- A single-database mapping (e.g., HMDB-only) is intended, which the paper warns can
  cause loss of information and mismatches
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaFetcheR
  role: Performs the queue-based identifier resolution and marks multi-mapped entries
    as ambiguous for user selection of the representative identifier
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: PostgreSQL
  role: Local cache database storing bulk-inserted dump data and on-the-fly fetched
    KEGG/PubChem instances used to detect multiple candidate compounds
- name: R
  role: Implementation language for the resolution loop (resolve_metabolites, resolve_single_id)
    and ambiguity flagging
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:b5145543198d0af7eceb7a87b7b2083f284dbdcd2dbb29407e86e89cb53d140b
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
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolomics-annotation-ambiguity-flagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Flags metabolite identifier entries that resolve to multiple candidate compounds from the same small-compound database (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS) during MetaFetcheR's queue-based resolution, rather than forcing a single mapping, and routes these ambiguous cases to the user for representative-identifier selection.

## When to use

Use when resolving metabolite identifiers across databases and an identifier maps to multiple candidate compounds from the same database, so ambiguity must be surfaced for user selection instead of silently forcing one mapping.

## When NOT to use

- The entity space is not small compounds (e.g., gene or protein identifiers) — MetaFetcheR covers only HMDB, ChEBI, PubChem, KEGG and LIPID MAPS
- Every metabolite already has a complete, unambiguous cross-database identifier set with no multi-mapped candidates
- A single-database mapping (e.g., HMDB-only) is intended, which the paper warns can cause loss of information and mismatches

## Inputs

- Reference metabolite table (df.discovered) containing known primary database identifiers with empty cross-database fields
- Database dump files from HMDB, ChEBI and LIPID MAPS (passed as HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name to install_databases)
- PostgreSQL cache database configured via db.connect(conf)

## Outputs

- resolved_metabolite_mappings.csv cross-database identifier mapping table
- Flagged ambiguous multi-mapped entries awaiting user selection of a representative identifier
- PostgreSQL cache instances newly fetched from KEGG and PubChem via HTTP

## How to apply

During resolve_metabolites(df.discovered), each record is filled via resolve_single_id(start_db_tag, start_db_id) using a local PostgreSQL cache populated by bulk insertion of HMDB, ChEBI and LIPID MAPS dump files (bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps) and on-the-fly http_call_api() retrieval from KEGG and PubChem via id_to_url(). The rationale is that missing foreign reference identifiers and overlapping database content commonly produce multiple plausible links, so the queue-based algorithm reiterates until identifiers are retrieved or cannot be further resolved. When a record maps to several candidate compounds within the same database, the entry is marked ambiguous rather than auto-resolved, and the user chooses the representative identifier that is then carried into downstream analysis (e.g., pathway enrichment). The final state is written to resolved_metabolite_mappings.csv with every identifier either resolved, user-selected, or exhausted.

## Related tools

- **MetaFetcheR** (Performs the queue-based identifier resolution and marks multi-mapped entries as ambiguous for user selection of the representative identifier) — https://github.com/komorowskilab/MetaFetcheR/
- **PostgreSQL** (Local cache database storing bulk-inserted dump data and on-the-fly fetched KEGG/PubChem instances used to detect multiple candidate compounds)
- **R** (Implementation language for the resolution loop (resolve_metabolites, resolve_single_id) and ambiguity flagging)

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher")
```

## Evaluation signals

- Every row of resolved_metabolite_mappings.csv is in one of three states: resolved, user-selected representative identifier, or exhausted/unresolvable
- Entries mapping to multiple candidate compounds from the same database are explicitly flagged as ambiguous, with no silently forced single mapping
- Cross-check a sample of resolved mappings against the source databases (e.g., HMDB-to-KEGG) to confirm the representative identifiers match user choices
- All newly fetched KEGG and PubChem records retrieved via http_call_api are present in the PostgreSQL cache after the run
- The resolution loop terminates only when all identifiers have been retrieved or cannot be further resolved

## Limitations

- Ambiguous multi-mapped entries require manual user selection of the representative identifier; no automatic disambiguation is provided
- Unique MetaFetcheR identifiers are only a planned future feature, not currently available
- The paper reports outperformance over MS_targeted and MetaboAnalyst without quantitative performance metrics
- Only five small-compound databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS) are supported; updates to include additional databases are planned

## Evidence
