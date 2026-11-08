---
name: quinonemethide-triterpene-annotation-retrieval
description: Use when an active extract's LC-MS/MS features are already annotated
  in the ENPKG knowledge graph and you need to recover known bioactive compounds (e.g.
  anti-T. cruzi activity in ChEMBL, target CHEMBL368) without re-isolation or re-screening.
when_to_use_negative:
- The sample has not been processed through the ENPKG workflow (MZmine feature detection,
  ISDB/SIRIUS annotation, RDF generation), so no annotated features exist to query
- The bioactivity of interest is not represented in ChEMBL or the integrated knowledge
  graph links, since no statistical metrics (FDR, p-values, effect sizes) back the
  annotations
- You need de novo structural elucidation of unknown compounds rather than recovery
  of previously reported (known) bioactive metabolites
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ENPKG
  role: Knowledge graph integrating sample-level LC-MS/MS annotations (ISDB, SIRIUS,
    NPClassifier) with ChEMBL and Wikidata links, queried via SPARQL
  repo: https://github.com/enpkg/enpkg_full
- name: GraphDB
  role: Hosts the RDF knowledge graph and the SPARQL research interface used to run
    the queries
  repo: https://enpkg.commons-lab.org/graphdb/
- name: SPARQL
  role: Query language used to traverse enpkg:has_wd_id, ISDB/SIRIUS annotations,
    NPClassifier classes, and ChEMBL bioactivity links
- name: ChEMBL
  role: Integrated bioactivity resource providing anti-T. cruzi activity links (target
    CHEMBL368) used to filter annotated compounds
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:ca71df125e8fd75946078931aa7de372b2deac9d73af1b0ce7d0f576f81f57aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_full
  tools:
  - ChEMBL
  - ENPKG
  - GraphDB
  - RDF
  - SPARQL
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# quinonemethide-triterpene-annotation-retrieval

## Summary

SPARQL query of the ENPKG knowledge graph (GraphDB endpoint) with integrated ChEMBL bioactivity links to rapidly recover known anti-T. cruzi compounds — e.g. the quinonemethide triterpenes pristimerin and zeylasteral — annotated in an active natural extract, validating hits against reported HRESIMS m/z values.

## When to use

Use when an active extract's LC-MS/MS features are already annotated in the ENPKG knowledge graph and you need to recover known bioactive compounds (e.g. anti-T. cruzi activity in ChEMBL, target CHEMBL368) without re-isolation or re-screening.

## When NOT to use

- The sample has not been processed through the ENPKG workflow (MZmine feature detection, ISDB/SIRIUS annotation, RDF generation), so no annotated features exist to query
- The bioactivity of interest is not represented in ChEMBL or the integrated knowledge graph links, since no statistical metrics (FDR, p-values, effect sizes) back the annotations
- You need de novo structural elucidation of unknown compounds rather than recovery of previously reported (known) bioactive metabolites

## Inputs

- ENPKG knowledge graph exposed as a GraphDB SPARQL endpoint (https://enpkg.commons-lab.org/graphdb/sparql)
- NPClassifier chemical taxonomy class of interest (e.g. quinonemethide triterpenes)
- ChEMBL bioactivity target identifier (T. cruzi, CHEMBL368)
- Biosource Wikidata identifier (Q11075650)
- Reported HRESIMS m/z values for candidate known compounds (465.3002, 495.2754)

## Outputs

- Result table of annotated known anti-T. cruzi compounds with compound name, Wikidata/ChEMBL ID, annotation type, and observed m/z
- Exported CSV (e.g. anti_tcruzi_showcase_results.csv)
- Structure–organism pairs retrieved via federated Wikidata/LOTUS queries (14 pairs in the showcase)

## How to apply

Open the ENPKG GraphDB endpoint (https://enpkg.commons-lab.org/graphdb/) and its SPARQL research interface at https://enpkg.commons-lab.org/graphdb/sparql (no login required). Write a SPARQL query that traverses enpkg:has_wd_id, ISDB/SIRIUS structural annotations, NPClassifier chemical taxonomy (e.g. quinonemethide triterpenes), and the integrated ChEMBL bioactivity links (T. cruzi, CHEMBL368), restricted to the biosource Wikidata node (Q11075650) — federated queries over multiple RDF endpoints are supported. Run the query and filter rows to structural candidates whose annotated feature masses, observed in positive ionization mode, match reported HRESIMS values (pristimerin 465.3002, zeylasteral 495.2754). Export matched rows (compound name, Wikidata/ChEMBL ID, annotation type, observed m/z) to a CSV such as anti_tcruzi_showcase_results.csv. Validation succeeds only if the result table contains rows for both pristimerin and zeylasteral with the expected m/z values recovered as known anti-T. cruzi compounds from the active extract.

## Related tools

- **ENPKG** (Knowledge graph integrating sample-level LC-MS/MS annotations (ISDB, SIRIUS, NPClassifier) with ChEMBL and Wikidata links, queried via SPARQL) — https://github.com/enpkg/enpkg_full
- **GraphDB** (Hosts the RDF knowledge graph and the SPARQL research interface used to run the queries) — https://enpkg.commons-lab.org/graphdb/
- **SPARQL** (Query language used to traverse enpkg:has_wd_id, ISDB/SIRIUS annotations, NPClassifier classes, and ChEMBL bioactivity links)
- **ChEMBL** (Integrated bioactivity resource providing anti-T. cruzi activity links (target CHEMBL368) used to filter annotated compounds)

## Examples

```
curl -H 'Accept: application/sparql-results+json' --data-urlencode 'query=SELECT * WHERE { ?s ?p ?o } LIMIT 10' https://enpkg.commons-lab.org/graphdb/sparql
```

## Evaluation signals

- Result table contains rows for both pristimerin and zeylasteral with m/z values matching the reported HRESIMS values 465.3002 and 495.2754 in positive ionization mode
- Retrieved compounds carry both a Wikidata ID (via enpkg:has_wd_id) and a ChEMBL ID with reported anti-T. cruzi activity (CHEMBL368)
- All returned compounds are consistent with the NPClassifier class filter (quinonemethide triterpenes) and the biosource restriction (Q11075650)
- Federated queries against Wikidata return documented structure–organism pairs (14 pairs in the showcase run)
- Exported CSV (anti_tcruzi_showcase_results.csv) contains 10 distinct anti-T. cruzi compounds with annotation type and observed m/z columns populated

## Limitations

- No statistical metrics (FDR, p-values, effect sizes, AUROC) are reported for the anti-T. cruzi compound annotations; retrieval confidence rests on spectral annotation and ChEMBL curation
- Merging experimental data with public resources (semantic-merging) has no reported false-positive/false-negative rate
- SPARQL/semantic web approaches are not yet democratized in natural products research, and no benchmark against existing workflows is provided

## Evidence

- [intro] SPARQL query language support: "These graphs can be queried using the SPARQL Protocol and RDF Query Language (SPARQL), a language designed to query and extract information from RDF databases."
- [discussion] Showcase result: "we could rapidly
annotate known anti-T. cruzi compounds
in an active extract, annotate unknown anti-T. cruzi compounds common to active extracts"
- [readme] Endpoint availability: "The ENPKG graph is available at the following address https://enpkg.commons-lab.org/graphdb/. No need for login !"
- [readme] SPARQL interface: "The SPARQL research interface can be reached at https://enpkg.commons-lab.org/graphdb/sparql. Make sure to check the [paper](https://doi.org/10.1021/acscentsci.3c00800) for examples of queries."
- [intro] Federated queries: "it is possible to elaborate federated queries over multiple end points to retrieve and link data from different RDF databases (DBs)"
