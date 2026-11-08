---
name: metabolite-catalog-curation
description: Use when metabolomic evidence for an organism is scattered across supplementary
  tables with mixed annotation states and you need a single indexed/orphan catalog
  reconciled against a genome-scale metabolic network (GSMN) producibility analysis.
when_to_use_negative:
- When the goal is to infer novel reactions or metabolite structures with PathModel's
  incremental ASP inference rather than curating an evidence-based catalog of already-detected
  metabolites.
- When all metabolite entries already share a single annotation source and no indexed/orphan
  split needs to be established.
- When the organism's GSMN producibility calls are unavailable — indexed metabolites
  could not be annotated with the producibility column, breaking the validation invariants.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: PathModel
  role: ASP-based framework whose knowledge base (encoded in .lp data files such as
    sterol_pwy.lp and MAA_pwy.lp) provides the pathway context into which the curated
    C. crispus metabolite catalog feeds
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder and solver used to execute the ASP programs underlying PathModel;
    must be installed with Lua compatibility
  repo: https://github.com/potassco/clingo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:8db35f5490cc175514aae906c923bfe4a7b990aa218b68847c9c1a25bd30b966
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pathmodel/pathmodel
  repo_url: https://github.com/pathmodel/pathmodel
  tools:
  - PathModel
  - clingo
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabolite-catalog-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a unified, evidence-based metabolite catalog for an organism of study (here, 142 experimentally evidenced Chondrus crispus metabolites from Supplementary Tables S1 and S2) by parsing entries into one dataframe and classifying each by annotation status (MetaCyc-indexed vs. orphan) and, for indexed entries, by GSMN producibility.

## When to use

Use when metabolomic evidence for an organism is scattered across supplementary tables with mixed annotation states and you need a single indexed/orphan catalog reconciled against a genome-scale metabolic network (GSMN) producibility analysis.

## When NOT to use

- When the goal is to infer novel reactions or metabolite structures with PathModel's incremental ASP inference rather than curating an evidence-based catalog of already-detected metabolites.
- When all metabolite entries already share a single annotation source and no indexed/orphan split needs to be established.
- When the organism's GSMN producibility calls are unavailable — indexed metabolites could not be annotated with the producibility column, breaking the validation invariants.

## Inputs

- Supplementary Table S1: experimentally evidenced metabolites already indexed in MetaCyc
- Supplementary Table S2: evidenced orphan metabolites not yet indexed in any database
- GSMN producibility analysis results for the organism (producible/unproducible calls per metabolite)

## Outputs

- c_crispus_metabolite_catalog.csv with columns metabolite name, annotation status (indexed/orphan), and GSMN producibility (producible/unproducible/NA for orphans)
- Indexation statistics: 142 total metabolites, 85 indexed vs. 57 orphan, 59 producible vs. 29 unproducible indexed metabolites

## How to apply

Parse the source supplementary tables (e.g. Tables S1 and S2 of the C. crispus study) to extract all experimentally evidenced metabolite entries into a single dataframe. Classify each entry by annotation status: 'indexed' if it already has a MetaCyc cross-referenced entry, 'orphan' if it has no database entry — in the reference study this yielded 85 indexed and 57 orphan metabolites (142 total). For indexed metabolites only, cross-reference against the organism's GSMN producibility analysis and record each as producible or unproducible (59 producible, 29 unproducible in the reference), leaving producibility as NA for orphans. Write the catalog to a CSV (e.g. c_crispus_metabolite_catalog.csv) with columns for metabolite name, annotation status, and GSMN producibility. Validate by checking the assembled table reproduces the reported counts: 142 total, 85/57 indexed/orphan split, and 59/29 producible/unproducible among indexed metabolites. ASP tooling from the paper (PathModel scripts run through the clingo grounder and solver) can supply the pathway-level context in which the catalog is later used for inference.

## Related tools

- **PathModel** (ASP-based framework whose knowledge base (encoded in .lp data files such as sterol_pwy.lp and MAA_pwy.lp) provides the pathway context into which the curated C. crispus metabolite catalog feeds) — https://github.com/pathmodel/pathmodel
- **clingo** (Grounder and solver used to execute the ASP programs underlying PathModel; must be installed with Lua compatibility) — https://github.com/potassco/clingo

## Evaluation signals

- Assembled table contains exactly 142 experimentally evidenced metabolites.
- Indexed/orphan classification reproduces the 85 indexed (MetaCyc) vs. 57 orphan split.
- Among the 85 indexed metabolites, producibility calls sum to 59 producible and 29 unproducible.
- Orphan metabolites carry producibility = NA (no GSMN cross-reference allowed for them).
- CSV schema check: columns are metabolite name, annotation status (indexed/orphan), GSMN producibility (producible/unproducible/NA).

## Limitations

- Manual burden: the starting knowledge base and metabolite evidence must be curated by hand; atom numbering and entry extraction are not fully automated.
- PathModel is a working prototype with open scalability issues, so the downstream use of the catalog is not production-grade.
- Producibility annotation is only meaningful for MetaCyc-indexed metabolites; orphans cannot be cross-referenced against the GSMN.

## Evidence

- [results] indexed vs orphan split: [evidence span withheld — anchor status: not_found]
- [methods] producibility cross-reference: [evidence span withheld — anchor status: not_found]
