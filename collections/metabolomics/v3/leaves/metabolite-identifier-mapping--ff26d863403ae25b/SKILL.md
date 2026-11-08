---
name: metabolite-identifier-mapping
description: Use when a metabolomics input table contains sparse primary database
  identifiers (e.g. HMDB, ChEBI, KEGG, PubChem, LIPID MAPS) with empty foreign-ID
  fields that must be filled for pathway enrichment or cross-database linkage.
when_to_use_negative:
- Input metabolites are already fully annotated with unique cross-database identifiers
  — mapping would be redundant (unique MetaFetcheR identifiers are only a planned
  future feature)
- The metabolite of interest is not covered by the five supported databases (HMDB,
  ChEBI, PubChem, KEGG, LIPID MAPS) — METLIN is mentioned in the article but bulk-cache
  insertion functions exist only for HMDB, ChEBI and LIPID MAPS
- The task requires a web-service mapper without local infrastructure — MetaFetcheR
  requires a populated local PostgreSQL cache to function
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0089
tools:
- name: MetaFetcheR
  role: 'Core R package: queue-based resolver (resolve_metabolites, resolve_single_id),
    PostgreSQL cache construction and HTTP on-the-fly fetching'
  repo: https://github.com/komorowskilab/MetaFetcheR/
- name: PostgreSQL
  role: Local cache database holding bulk-inserted HMDB/ChEBI/LIPID MAPS dumps and
    on-the-fly KEGG/PubChem records
- name: R
  role: Implementation language; installed via devtools::install_github with helper
    package shizidushu/hfun
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:e06284edc1f54e6d9b600704e76903bf53c5366e14d0a724495598ea70c11601
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
  - ChemmineR
  - KEGGREST
  - MetaFetcheR
  - MetaMapR
  - PostgreSQL
  - PubChem Substructure Fingerprints
  - R
  - chebi
  - hmdb
  - kegg
  - lipidmaps
  - omu
  - pubchem
  - read.metabo
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# metabolite-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps metabolite identifiers across five small-compound databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS) using MetaFetcheR, a queue-based algorithm backed by a local PostgreSQL cache that reiterates exhaustively until identifiers are resolved or cannot be further resolved.

## When to use

Use when a metabolomics input table contains sparse primary database identifiers (e.g. HMDB, ChEBI, KEGG, PubChem, LIPID MAPS) with empty foreign-ID fields that must be filled for pathway enrichment or cross-database linkage.

## When NOT to use

- Input metabolites are already fully annotated with unique cross-database identifiers — mapping would be redundant (unique MetaFetcheR identifiers are only a planned future feature)
- The metabolite of interest is not covered by the five supported databases (HMDB, ChEBI, PubChem, KEGG, LIPID MAPS) — METLIN is mentioned in the article but bulk-cache insertion functions exist only for HMDB, ChEBI and LIPID MAPS
- The task requires a web-service mapper without local infrastructure — MetaFetcheR requires a populated local PostgreSQL cache to function

## Inputs

- Sparse metabolite identifier table (data frame, e.g. df.discovered) with known primary database identifiers and empty foreign-ID fields
- HMDB, ChEBI and LIPID MAPS database dump files for populating the PostgreSQL cache
- PostgreSQL connection parameters (host, port, db_name, user, password, path)

## Outputs

- Resolved cross-database identifier mapping table (e.g. resolved_metabolite_mappings.csv)
- Non-empty-field mapping rate per dataset (e.g. mapping_rate_diamanti.csv, mapping_rate_priolo2014.csv)
- Cached KEGG/PubChem records stored in the local PostgreSQL database

## How to apply

Install MetaFetcheR and its helper package in R via install_github("shizidushu/hfun","komorowskilab/metafetcher"), then construct a local PostgreSQL cache: download HMDB, ChEBI and LIPID MAPS dump files and bulk-insert them with bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps and bulk_insert_secondary_ids, after configuring with write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name) and connecting via db.connect(conf). KEGG and PubChem records are fetched on the fly through http_call_api with URLs built by id_to_url(db_id, db_tag) and stored back into the cache. Load the sparse identifier table as df.discovered and run resolve_metabolites(df.discovered); the queue-based loop fills empty identifier fields (checked with null2na(v) and is.empty(v)) by reiterating exhaustively until all identifiers are retrieved or cannot be further resolved, writing ambiguous multi-mapped values as joined arrays via join_sql_arr(v) inside db.transaction/db.commit cycles. Ambiguous entries that map to multiple candidate compounds are flagged for user selection of a representative identifier rather than forcing a single mapping. Report the mapping rate as the fraction of resolved (non-empty) identifier fields over input entries.

## Related tools

- **MetaFetcheR** (Core R package: queue-based resolver (resolve_metabolites, resolve_single_id), PostgreSQL cache construction and HTTP on-the-fly fetching) — https://github.com/komorowskilab/MetaFetcheR/
- **PostgreSQL** (Local cache database holding bulk-inserted HMDB/ChEBI/LIPID MAPS dumps and on-the-fly KEGG/PubChem records)
- **R** (Implementation language; installed via devtools::install_github with helper package shizidushu/hfun)

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher"); conf <- write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name); db.connect(conf); resolved <- resolve_metabolites(df.discovered)
```

## Evaluation signals

- On the Diamanti et al. 2019 dataset the non-empty-field mapping rate should be ~80%, versus ~48% for MetaboAnalyst (Figure 1B reference)
- On the Priolo et al. 2014 dataset the non-empty-field mapping rate should be ~78%, versus ~62% for MetaboAnalyst (Figure 1C reference)
- Consistency test via do_consistency_test(db, n=1000, attempts=100) should reproduce Supplementary Tables S2-S6 rates, e.g. KEGG ~65% consistency / ~23% unresolved and HMDB ~71% unresolved
- No identifier fields remain empty for records that can be further resolved after the queue exhaustively reiterates
- Ambiguous multi-mapped entries are stored as joined arrays of multiple identifiers, flagged for user selection rather than silently collapsed

## Limitations

- HMDB had the highest fraction of unresolved cases (~71%) in the quality test, so many HMDB-started resolutions may remain incomplete
- Mapping rate depends on cache completeness: KEGG and PubChem require live HTTP calls, so coverage and reproducibility depend on network access and cache state
- No version number is stated for MetaFetcheR and unique MetaFetcheR identifiers are only a planned future update
- Benchmark datasets are cited only by publication reference (Diamanti et al., 2019; Priolo et al., 2014) with no accession or data availability information in the article
- Existing tools' reliance on a lacking shared nomenclature means mismatches or no-matches may persist for names that cannot be resolved

## Evidence
