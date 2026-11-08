---
name: chembl-bioactivity-metadata-linking
description: Use when a bioactive natural extract's annotated features should be prioritized
  by checking whether known structures in the KG already have reported activity against
  the screened organism, e.g. anti-T. cruzi quinoline alkaloids in Waltheria indica
  extracts.
when_to_use_negative:
- The input is raw LC-MS data or a feature quantification table that has not yet been
  processed into ENPKG RDF annotations — ChEMBL linkage requires isdb/SIRIUS annotations
  and Wikidata IDs first
- The target organism or assay has no ChEMBL/Wikidata representation in the integrated
  KG, so no bioactivity edges can be traversed
- When novel-compound prioritization without prior activity reports is the goal —
  known-activity joins only recover documented pairs, so purely unknown features fall
  outside this skill.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: SPARQL
  role: Query language used to traverse the ENPKG RDF graph and join sample annotations
    to ChEMBL bioactivity records
- name: ENPKG
  role: Knowledge-graph framework whose GraphDB endpoint hosts the integrated ChEMBL/Wikidata
    layer queried here
  repo: https://github.com/enpkg/enpkg_full
- name: GraphDB
  role: RDF store exposing the ENPKG graph and its SPARQL research interface
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:7227e1a87b8e3c687c2bb06f11fea7a4dd21939bef294593db791d999b7822a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_full
  tools:
  - ENPKG
  - GraphDB
  - RDF
  - SPARQL
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chembl-bioactivity-metadata-linking

## Summary

Query the ENPKG knowledge graph's integrated ChEMBL layer via SPARQL to surface compounds annotated in a sample that carry reported activity against a target organism, connecting structural annotations (isdb/SIRIUS, Wikidata IDs) to bioactivity records such as T. cruzi (CHEMBL368).

## When to use

Use when a bioactive natural extract's annotated features should be prioritized by checking whether known structures in the KG already have reported activity against the screened organism, e.g. anti-T. cruzi quinoline alkaloids in Waltheria indica extracts.

## When NOT to use

- The input is raw LC-MS data or a feature quantification table that has not yet been processed into ENPKG RDF annotations — ChEMBL linkage requires isdb/SIRIUS annotations and Wikidata IDs first
- The target organism or assay has no ChEMBL/Wikidata representation in the integrated KG, so no bioactivity edges can be traversed
- When novel-compound prioritization without prior activity reports is the goal — known-activity joins only recover documented pairs, so purely unknown features fall outside this skill.

## Inputs

- RDF knowledge graph (.ttl files) loaded in GraphDB, with per-sample feature annotations, enpkg:has_wd_id links, NPClassifier classes, and integrated ChEMBL bioactivity edges
- Sample metadata (taxon/biosource Wikidata Q-ID) for the active extract
- Target organism of interest (e.g. T. cruzi / CHEMBL368)

## Outputs

- SPARQL result table of annotated compounds with reported activity against the target organism (compound name, Wikidata/ChEMBL ID, annotation type, observed m/z)
- CSV export of matched structure–organism pairs (e.g. anti_tcruzi_showcase_results.csv)

## How to apply

Write a SPARQL query against the ENPKG GraphDB endpoint (https://enpkg.commons-lab.org/graphdb/sparql) traversing enpkg:has_wd_id links from annotated features, then joining Wikidata/ChEMBL bioactivity records for the target organism — in the paper, T. cruzi, ChEMBL target CHEMBL368 — filtered to the sample's biosource (e.g. Pristimera indica, Q11075650, or Waltheria indica for the showcase). Include NPClassifier taxonomy predicates (e.g. quinonemethide triterpenes) to characterize chemical classes, and cross-check returned compounds against observed feature masses in the relevant ionization mode (paper: pristimerin 465.3002, zeylasteral 495.2754 in positive mode HRESIMS). Export matched rows (compound name, Wikidata/ChEMBL ID, annotation type, observed m/z) to a table such as anti_tcruzi_showcase_results.csv. The query exploits the KG's sample-centric RDF design, in which each sample's annotations were integrated with public knowledge (ChEMBL, LOTUS, Wikidata) precisely so such federated joins can prioritize known bioactives and flag novel analogs.

## Related tools

- **SPARQL** (Query language used to traverse the ENPKG RDF graph and join sample annotations to ChEMBL bioactivity records)
- **ENPKG** (Knowledge-graph framework whose GraphDB endpoint hosts the integrated ChEMBL/Wikidata layer queried here) — https://github.com/enpkg/enpkg_full
- **GraphDB** (RDF store exposing the ENPKG graph and its SPARQL research interface)

## Examples

```
curl -G https://enpkg.commons-lab.org/graphdb/sparql --data-urlencode 'query=PREFIX enpkg: <https://enpkg.commons-lab.org/kg/> SELECT ?compound ?wd ?mz WHERE { ?f enpkg:has_wd_id ?wd . ?wd wdt:P682 ?org . VALUES ?org { wd:Q30502 } }'
```

## Evaluation signals

- Result table contains rows for the expected known actives (paper validation: pristimerin and zeylasteral present)
- Observed m/z values for matched compounds agree with reported HRESIMS masses (465.3002 and 495.2754 in positive ionization mode)
- Each returned compound carries a resolvable Wikidata/ChEMBL identifier and an annotation type (isdb or SIRIUS)
- Reported structure–organism pairs from Wikidata are counted and consistent with the query scope (paper: 14 pairs, 10 distinct compounds)
- Query results are restricted to the specified biosource Q-ID (e.g. Q11075650), confirming correct sample-level filtering

## Limitations

- No statistical confidence metrics (FDR, p-values, AUROC) are reported for the anti-T. cruzi compound annotations recovered via these joins
- Only compounds already documented in ChEMBL/Wikidata with activity against the queried organism will surface; novel actives require the separate analog-retrieval workflow
- Ties depend on the currency of the integrated public resources (ChEMBL, LOTUS, Wikidata) at graph-build time; the KG is not automatically re-linked when those sources update.

## Evidence

- [results] SPARQL query recovering known anti-T. cruzi annotated compounds: "we could rapidly annotate known anti-T. cruzi compounds in an active extract, annotate unknown anti-T. cruzi compounds common to active extracts"
- [results] SPARQL join of Wikidata structure-organism pairs to ChEMBL target: [evidence span withheld — anchor status: not_found]
- [results] ENPKG integrates ChEMBL bioactivity into the graph for SPARQL querying: [evidence span withheld — anchor status: not_found]
- [results] GraphDB endpoint and SPARQL interface for querying ENPKG: "The ENPKG graph is available at the following address https://enpkg.commons-lab.org/graphdb/. No need for login !"
