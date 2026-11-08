---
name: metabolite-identifier-mapping
description: Use when building a literature-derived metabolite catalog from supplementary
  tables and mapping each entry against database indexation (e.g., MetaCyc) and GSMN
  producibility to quantify indexed vs. orphan coverage.
when_to_use_negative:
- The goal is de novo prediction of biochemical reactions or metabolite structures
  — use PathModel's inference mode instead of identifier classification
- Input tables are mass-spectrometry feature lists (m/z values) rather than named
  metabolite entries; m/z filtering in PathModel (M/Z ratio multiplied by 10,000)
  is a separate concern
- Metabolites lack experimental evidence of presence — this skill catalogs evidenced
  compounds only
tools:
- name: PathModel
  role: ASP-based framework within which the classified C. crispus metabolite catalog
    is assembled and used; prototype underlying the paper's analyses
  repo: github.com/pathmodel/pathmodel
- name: clingo
  role: Grounder and solver for the Answer Set Programming scripts (ReactionSiteExtraction.lp,
    MZComputation.lp, PathModel.lp) that consume the metabolite data
  repo: https://github.com/potassco/clingo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:f661f772edad50c005205bceb96ea62ade215f4272e5abc6888cb7865827b145
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pathmodel/pathmodel
  tools:
  - PathModel
  - clingo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metabolite-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles an experimentally evidenced metabolite catalog for Chondrus crispus by parsing Supplementary Tables S1 and S2, then classifies each entry by annotation status (MetaCyc-indexed vs. orphan) and by genome-scale metabolic network (GSMN) producibility.

## When to use

Use when building a literature-derived metabolite catalog from supplementary tables and mapping each entry against database indexation (e.g., MetaCyc) and GSMN producibility to quantify indexed vs. orphan coverage.

## When NOT to use

- The goal is de novo prediction of biochemical reactions or metabolite structures — use PathModel's inference mode instead of identifier classification
- Input tables are mass-spectrometry feature lists (m/z values) rather than named metabolite entries; m/z filtering in PathModel (M/Z ratio multiplied by 10,000) is a separate concern
- Metabolites lack experimental evidence of presence — this skill catalogs evidenced compounds only

## Inputs

- Supplementary Table S1 (experimentally evidenced C. crispus metabolites already indexed in MetaCyc)
- Supplementary Table S2 (experimentally evidenced C. crispus orphan metabolites not yet indexed)
- C. crispus GSMN producibility analysis results (for indexed metabolites)

## Outputs

- c_crispus_metabolite_catalog.csv with columns: metabolite name, annotation status (indexed/orphan), GSMN producibility (producible/unproducible/NA for orphans)

## How to apply

Parse Supplementary Tables S1 and S2 into a single dataframe containing all experimentally evidenced C. crispus metabolite entries. Classify each metabolite by annotation status: MetaCyc-indexed entries (cross-referenced, Table S1) versus orphans lacking a database entry (Table S2), reproducing the reported 85 indexed / 57 orphan split (142 total). For indexed metabolites only, cross-reference against the C. crispus GSMN producibility analysis and record each as producible (59) or unproducible (29); orphans receive NA for producibility. Write the catalog to a CSV with columns for metabolite name, annotation status (indexed/orphan), and GSMN producibility (producible/unproducible/NA). Validate that the assembled table reproduces the published statistics before using it downstream in PathModel/GSMN analyses.

## Related tools

- **PathModel** (ASP-based framework within which the classified C. crispus metabolite catalog is assembled and used; prototype underlying the paper's analyses) — github.com/pathmodel/pathmodel
- **clingo** (Grounder and solver for the Answer Set Programming scripts (ReactionSiteExtraction.lp, MZComputation.lp, PathModel.lp) that consume the metabolite data) — https://github.com/potassco/clingo

## Evaluation signals

- Assembled table contains exactly 142 total evidenced metabolites
- Indexed/orphan annotation split reproduces 85 MetaCyc-indexed (Table S1) and 57 orphan (Table S2) entries
- Among the 85 indexed metabolites, producibility counts are 59 producible and 29 unproducible
- All orphan metabolites carry NA in the GSMN producibility column
- CSV schema contains the three required columns: metabolite name, annotation status, GSMN producibility

## Limitations

- Orphan metabolites (57 of 142) have no database entry and therefore cannot receive a GSMN producibility status (recorded as NA)
- Indexation status reflects the MetaCyc snapshot at the time of analysis; orphans may be indexed later
- The surrounding PathModel tooling is explicitly a working prototype with no guarantee of correctness, which constrains downstream reuse of the catalog

## Evidence

- [supplementary] Two-category division of evidenced metabolites: [evidence span withheld — anchor status: not_found]
- [results] Validation statistics for the assembled catalog: [evidence span withheld — anchor status: not_found]
