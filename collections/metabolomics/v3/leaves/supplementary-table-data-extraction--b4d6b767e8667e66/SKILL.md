---
name: supplementary-table-data-extraction
description: Use when Supplementary Tables contain curated entity lists with database
  cross-references (e.g. MetaCyc-indexed vs. orphan metabolites) that must be merged
  into one dataset and validated against reported indexation statistics.
when_to_use_negative:
- The catalog already exists as a structured machine-readable file — re-parsing supplementary
  tables would duplicate curation effort.
- The goal is to run PathModel's ASP inference itself (pathmodel infer over .lp files)
  rather than assembling the evidenced-metabolite inventory that feeds it.
- The supplementary material is only available as raster images/figures without machine-readable
  table text, so faithful extraction cannot be guaranteed.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: PathModel
  role: Framework whose MetaCyc-derived knowledge base and GSMN producibility context
    the extracted catalog supports and validates against
  repo: https://github.com/pathmodel/pathmodel
- name: clingo
  role: ASP grounder and solver underlying PathModel, which consumes the encoded metabolite
    knowledge referenced by the catalog
  repo: https://github.com/potassco/clingo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.100849
    title: Inferring Biochemical Reactions and Metabolite Structures to Understand
      Metabolic Pathway Drift
  source_content_hash: sha256:e67dacac5cd5d251665e9e1d606d1af25083a7406b48e3114cadb05d814c3602
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# supplementary-table-data-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a curated metabolite catalog by parsing a paper's Supplementary Tables (here Tables S1 and S2 of the C. crispus metabolic drift study), classifying each entry by database annotation status and cross-referencing it against genome-scale metabolic network (GSMN) producibility results.

## When to use

Use when Supplementary Tables contain curated entity lists with database cross-references (e.g. MetaCyc-indexed vs. orphan metabolites) that must be merged into one dataset and validated against reported indexation statistics.

## When NOT to use

- The catalog already exists as a structured machine-readable file — re-parsing supplementary tables would duplicate curation effort.
- The goal is to run PathModel's ASP inference itself (pathmodel infer over .lp files) rather than assembling the evidenced-metabolite inventory that feeds it.
- The supplementary material is only available as raster images/figures without machine-readable table text, so faithful extraction cannot be guaranteed.

## Inputs

- Supplementary Table S1 (database metabolites already indexed in MetaCyc)
- Supplementary Table S2 (orphan metabolites not yet indexed)
- C. crispus GSMN producibility analysis results (producible/unproducible per indexed metabolite)

## Outputs

- c_crispus_metabolite_catalog.csv with columns: metabolite name, annotation status (indexed/orphan), GSMN producibility (producible/unproducible/NA for orphans)
- Indexation statistics: 142 total evidenced metabolites, 85/57 indexed/orphan split, 59/29 producible/unproducible counts for indexed metabolites

## How to apply

Parse both supplementary tables into a single dataframe: Table S1 holds experimentally evidenced C. crispus metabolites already indexed in MetaCyc (85 entries), Table S2 holds evidenced orphan metabolites not yet indexed anywhere (57 entries). Classify each row by annotation status (indexed vs. orphan), then join the indexed subset against the C. crispus GSMN producibility analysis, recording 59 producible and 29 unproducible indexed metabolites; orphans receive NA for producibility. Write the result to c_crispus_metabolite_catalog.csv with columns for metabolite name, annotation status (indexed/orphan), and GSMN producibility (producible/unproducible/NA). The rationale is that the article's integrative genomic-plus-metabolomic approach depends on an accurate evidenced-metabolite inventory before PathModel/clingo-based inference, so the catalog must reproduce the published 142/85/57 and 59/29 statistics exactly.

## Related tools

- **PathModel** (Framework whose MetaCyc-derived knowledge base and GSMN producibility context the extracted catalog supports and validates against) — https://github.com/pathmodel/pathmodel
- **clingo** (ASP grounder and solver underlying PathModel, which consumes the encoded metabolite knowledge referenced by the catalog) — https://github.com/potassco/clingo

## Examples

```
python -c "import pandas as pd; cat = pd.concat([pd.read_excel('Table_S1.xlsx'), pd.read_excel('Table_S2.xlsx')]); cat.to_csv('c_crispus_metabolite_catalog.csv', index=False)"
```

## Evaluation signals

- Assembled table contains exactly 142 evidenced metabolite rows.
- Indexed/orphan split reproduces 85 (Table S1) vs. 57 (Table S2).
- Within indexed metabolites, producibility counts are 59 producible and 29 unproducible.
- Every orphan-metabolite row has GSMN producibility set to NA, never producible/unproducible.
- Output schema check: columns for metabolite name, annotation status, and GSMN producibility are present and non-null in c_crispus_metabolite_catalog.csv.

## Limitations

- The reported statistics (142 total, 85/57, 59/29) are specific to the C. crispus dataset; other organisms require re-deriving the counts rather than asserting them.
- Catalog completeness is bounded by the curation quality of the supplementary tables and the manual effort of encoding the starting knowledge base, which the authors flag as the main limitation.
- PathModel is described as a working prototype with open scalability issues, so downstream use of the extracted catalog inherits that maturity level.

## Evidence

- [results] Two-category split of evidenced metabolites by MetaCyc annotation status: [evidence span withheld — anchor status: not_found]
- [results] Producibility cross-reference counts for indexed metabolites: [evidence span withheld — anchor status: not_found]
- [discussion] Prototype status and scalability caveat: [evidence span withheld — anchor status: not_found]
