---
name: mapping-rate-benchmark-comparison
description: Use when you need to quantify how completely a metabolite identifier
  mapper fills HMDB/ChEBI/PubChem/KEGG/LIPID MAPS fields on a sparse identifier table,
  and compare that rate against a reference tool's reported rate.
when_to_use_negative:
- When the goal is pathway enrichment analysis itself rather than measuring identifier-mapping
  completeness — mapping to a predefined set of unique identifiers is a prerequisite
  step, not the benchmark.
- When the input table is already fully resolved (no empty identifier fields), since
  the queue-based reiteration and non-empty-field metric have nothing to resolve.
- When comparing against MS_targeted rather than MetaboAnalyst, since the concrete
  reference rates here (~80/48% and ~78/62%) come only from the MetaboAnalyst comparisons
  in Figures 1B–1C.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3391
tools:
- name: MetaFetcheR
  role: R package whose queue-based resolve_metabolites algorithm fills empty identifier
    fields across HMDB, ChEBI, PubChem, KEGG and LIPID MAPS; its resulting non-empty-field
    rate is the benchmarked metric
  repo: github.com/komorowskilab/MetaFetcheR
- name: PostgreSQL
  role: Local cache database constructed from HMDB, ChEBI and LIPID MAPS dump files
    via bulk insertion, serving identifier lookups during resolution
- name: MetaboAnalyst
  role: Reference mapping tool whose reported mapping rates (~48% Diamanti 2019, ~62%
    Priolo 2014) are used as the comparison baseline
- name: R
  role: Implementation language; MetaFetcheR is installed via devtools::install_github
    and the mapping rate is computed with its null2na and is.empty helpers
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1101/2021.02.28.433248
    title: 'MetaFetcheR: An R package for complete mapping of small compound data'
  source_content_hash: sha256:6ec7134e681811b529f680602bc4747b5d7953467c1c34c54cc380d29c8ea7bc
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
  - MetaboAnalyst
  - PostgreSQL
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# mapping-rate-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks MetaFetcheR's queue-based metabolite identifier resolution against an existing mapper (MetaboAnalyst) by computing the non-empty-field mapping rate on published metabolomics identifier tables (Diamanti et al. 2019; Priolo et al. 2014) and comparing the result to the reference rates reported in the paper's Figures 1B–1C.

## When to use

Use when you need to quantify how completely a metabolite identifier mapper fills HMDB/ChEBI/PubChem/KEGG/LIPID MAPS fields on a sparse identifier table, and compare that rate against a reference tool's reported rate.

## When NOT to use

- When the goal is pathway enrichment analysis itself rather than measuring identifier-mapping completeness — mapping to a predefined set of unique identifiers is a prerequisite step, not the benchmark.
- When the input table is already fully resolved (no empty identifier fields), since the queue-based reiteration and non-empty-field metric have nothing to resolve.
- When comparing against MS_targeted rather than MetaboAnalyst, since the concrete reference rates here (~80/48% and ~78/62%) come only from the MetaboAnalyst comparisons in Figures 1B–1C.

## Inputs

- Sparse metabolite identifier table from a published metabolomics study (e.g. Diamanti et al. 2019 or Priolo et al. 2014) loaded as df.discovered
- Database dump files for HMDB, ChEBI and LIPID MAPS
- Local PostgreSQL cache configuration parameters (host, port, db_name, user, password, path, dump file names)
- Reference mapping rates from the article's Figures 1B–1C (MetaboAnalyst and MetaFetcheR)

## Outputs

- Computed non-empty-field mapping rate for MetaFetcheR on the input identifier table
- CSV file (e.g. mapping_rate_diamanti.csv or mapping_rate_priolo2014.csv) containing the computed rate and both reference rates

## How to apply

Install MetaFetcheR from GitHub with install_github("shizidushu/hfun","komorowskilab/metafetcher"), then build a local PostgreSQL cache: download HMDB, ChEBI and LIPID MAPS dump files, configure with write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name), connect with db.connect(conf), and populate via bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps and bulk_insert_secondary_ids; KEGG and PubChem records are cached on the fly via http_call_api. Load the sparse metabolite identifier table (e.g. Diamanti et al. 2019 or Priolo et al. 2014) as df.discovered and run resolve_metabolites(df.discovered), letting the queue-based algorithm reiterate exhaustively until all identifiers are retrieved or cannot be further resolved. Compute the mapping rate as the fraction of resolved identifier fields that are non-empty, using the null2na and is.empty helpers. Write the computed rate together with the paper's reference rates (~80% vs ~48% MetaboAnalyst for Diamanti 2019, Figure 1B; ~78% vs ~62% for Priolo 2014, Figure 1C) to a CSV, and judge correctness by expert review of closeness to the reference value, since no exact threshold is stated.

## Related tools

- **MetaFetcheR** (R package whose queue-based resolve_metabolites algorithm fills empty identifier fields across HMDB, ChEBI, PubChem, KEGG and LIPID MAPS; its resulting non-empty-field rate is the benchmarked metric) — github.com/komorowskilab/MetaFetcheR
- **PostgreSQL** (Local cache database constructed from HMDB, ChEBI and LIPID MAPS dump files via bulk insertion, serving identifier lookups during resolution)
- **MetaboAnalyst** (Reference mapping tool whose reported mapping rates (~48% Diamanti 2019, ~62% Priolo 2014) are used as the comparison baseline)
- **R** (Implementation language; MetaFetcheR is installed via devtools::install_github and the mapping rate is computed with its null2na and is.empty helpers)

## Examples

```
library(devtools); install_github("shizidushu/hfun","komorowskilab/metafetcher"); conf <- write_config(host, port, db_name, user, password, path, HMDB_file_name, ChEBI_file_name, LIPIDMAPS_file_name); db.connect(conf); resolve_metabolites(df.discovered)
```

## Evaluation signals

- Script runs to completion: PostgreSQL cache is populated from all three dump files (bulk_insert_hmdb, bulk_insert_chebi, bulk_insert_lipidmaps) and resolve_metabolites terminates after exhaustive reiteration.
- Output CSV contains three values: the computed MetaFetcheR mapping rate plus both reference rates from Figures 1B–1C.
- Computed mapping rate on Diamanti et al. 2019 is consistent with the ~80% MetaFetcheR / ~48% MetaboAnalyst reference (expert review of closeness, no exact threshold stated).
- Computed mapping rate on Priolo et al. 2014 is consistent with the ~78% MetaFetcheR / ~62% MetaboAnalyst reference from Figure 1C.
- Every empty identifier field reachable through cross-database links is resolved before the queue-based algorithm declares identifiers unretrievable (checked with is.empty after null2na conversion).

## Limitations

- The paper reports no exact acceptance threshold for agreement with the reference rates; validation is expert review of closeness.
- Benchmark datasets (Diamanti et al. 2019; Priolo et al. 2014) are cited only by publication reference, with no accession or data availability information.
- Mapping relies on KEGG and PubChem data fetched over HTTP on the fly, so results depend on live API availability and the completeness of the local dump-file imports.
- A unique MetaFetcheR identifier was only a planned future feature, not available at publication time, so cross-database linkage rests on the five external identifier sets.

## Evidence
