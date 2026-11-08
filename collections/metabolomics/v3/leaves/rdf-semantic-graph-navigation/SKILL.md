---
name: rdf-semantic-graph-navigation
description: Use when processed metabolomics features are already integrated as RDF
  (e.g. ENPKG .ttl in GraphDB) and you need to answer questions spanning annotations,
  taxonomy, and external knowledge bases like Wikidata, LOTUS or ChEMBL.
when_to_use_negative:
- Input is raw LC-MS/MS DDA data or a feature table not yet processed and converted
  to RDF — run the ENPKG workflow (MZmine feature detection, taxonomy resolution via
  Open Tree of Life, ISDB/SIRIUS annotation) first
- The question only requires spectral similarity or molecular networking on a single
  dataset without integration of external knowledge bases (Wikidata, LOTUS, ChEMBL)
- Retention-time-based feature alignment across a single homogeneous batch is sufficient
  — a classical FBMN pipeline avoids the overhead of semantic querying
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3361
tools:
- name: ENPKG
  role: Provides the Experimental Natural Products Knowledge Graph (RDF) that is queried;
    integrates sample-level annotations with Wikidata and ChEMBL
  repo: https://github.com/enpkg/enpkg_full
- name: SPARQL
  role: Query language used to traverse the RDF knowledge graph, including federated
    queries over multiple endpoints
- name: RDF
  role: Standard graph model in which standardized experimental data and public knowledge
    (LOTUS/Wikidata structure-organism pairs) are represented
- name: GraphDB
  role: Hosts the public ENPKG SPARQL endpoint used to explore the generated knowledge
    graph
  repo: https://enpkg.commons-lab.org/graphdb/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:bc6183081d44c56338e1c317f42b6877dafe84884582d377f17925563e8b35cf
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# rdf-semantic-graph-navigation

## Summary

Formulate SPARQL queries against the ENPKG RDF knowledge graph (GraphDB endpoint) to traverse sample-level LC-MS/MS annotations, NPClassifier taxonomy, and integrated Wikidata/LOTUS and ChEMBL links, enabling knowledge-driven compound recovery such as retrieving annotated anti-T. cruzi compounds from active extracts.

## When to use

Use when processed metabolomics features are already integrated as RDF (e.g. ENPKG .ttl in GraphDB) and you need to answer questions spanning annotations, taxonomy, and external knowledge bases like Wikidata, LOTUS or ChEMBL.

## When NOT to use

- Input is raw LC-MS/MS DDA data or a feature table not yet processed and converted to RDF — run the ENPKG workflow (MZmine feature detection, taxonomy resolution via Open Tree of Life, ISDB/SIRIUS annotation) first
- The question only requires spectral similarity or molecular networking on a single dataset without integration of external knowledge bases (Wikidata, LOTUS, ChEMBL)
- Retention-time-based feature alignment across a single homogeneous batch is sufficient — a classical FBMN pipeline avoids the overhead of semantic querying

## Inputs

- RDF .ttl knowledge graph loaded in GraphDB (ENPKG repository-scale graph of 1,600 plant extract samples plus 337 Korean medicinal plants)
- SPARQL query traversing sample-level annotations, NPClassifier classes, Wikidata IDs (via enpkg:has_wd_id), and ChEMBL bioactivity links
- Optional biosource Wikidata QID (e.g. Q11075650) and reported HRESIMS m/z values for validation

## Outputs

- SPARQL result table of annotated compounds with structure-organism pairs from Wikidata/LOTUS and ChEMBL bioactivity links
- Exported CSV of matched rows (compound name, Wikidata/ChEMBL ID, annotation type, observed m/z), e.g. anti_tcruzi_showcase_results.csv

## How to apply

Open the ENPKG GraphDB SPARQL research interface at https://enpkg.commons-lab.org/graphdb/sparql and write a SPARQL query that traverses the graph predicates (e.g. enpkg:has_wd_id, ISDB/SIRIUS structural annotations, NPClassifier chemical classes such as quinonemethide triterpenes or quinoline alkaloids) and, where needed, federates to ChEMBL bioactivity endpoints (e.g. target T. cruzi, CHEMBL368) restricted to a biosource QID such as Q11075650. The rationale is that each sample was processed individually and semantically enriched into a unified KG, so a single query replaces retention-time-based feature alignment across batches. Run the query, then validate candidate matches against orthogonal evidence: compare annotated feature m/z values in positive ionization mode against reported HRESIMS values (e.g. 465.3002 and 495.2754 for known anti-T. cruzi compounds). Export matched rows (compound name, Wikidata/ChEMBL ID, annotation type, observed m/z) to a CSV such as anti_tcruzi_showcase_results.csv for downstream curation.

## Related tools

- **ENPKG** (Provides the Experimental Natural Products Knowledge Graph (RDF) that is queried; integrates sample-level annotations with Wikidata and ChEMBL) — https://github.com/enpkg/enpkg_full
- **SPARQL** (Query language used to traverse the RDF knowledge graph, including federated queries over multiple endpoints)
- **RDF** (Standard graph model in which standardized experimental data and public knowledge (LOTUS/Wikidata structure-organism pairs) are represented)
- **GraphDB** (Hosts the public ENPKG SPARQL endpoint used to explore the generated knowledge graph) — https://enpkg.commons-lab.org/graphdb/

## Examples

```
curl -G "https://enpkg.commons-lab.org/graphdb/sparql" --data-urlencode "query=SELECT ?wd_id ?mz WHERE { ?f a enpkg:Feature ; enpkg:has_wd_id ?wd_id . ?f enpkg:mz ?mz }"
```

## Evaluation signals

- Result rows contain the expected known compounds (e.g. pristimerin and zeylasteral) with Wikidata/ChEMBL IDs resolved via enpkg:has_wd_id
- Observed positive-ionization m/z values of matched annotated features match reported HRESIMS values (465.3002 and 495.2754) within instrument tolerance
- Structure-organism pairs returned from federated Wikidata/LOTUS queries are consistent with the literature-reported biosource of each compound
- Query on the showcase use case returns 10 distinct anti-T. cruzi compounds and 14 structure-organism pairs from the active extract's biosource
- Exported CSV contains complete columns (compound name, Wikidata/ChEMBL ID, annotation type, observed m/z) with no unmatched validation candidates

## Limitations

- No statistical metrics (e.g. FDR, p-values, effect sizes, AUROC) were reported for the anti-T. cruzi / anti-L. donovani compound annotations, so query results require orthogonal validation (e.g. m/z matching)
- Merging experimental data with public resources may introduce unreported false positives/false negatives from semantic merging
- The KG contains only features with an associated MS2 scan and RT between 0.5 and 8.0 min after filtering, so queries cannot recover filtered-out features

## Evidence

- [intro] SPARQL queries over the ENPKG graph are the mechanism for knowledge-driven recovery of annotated compounds from active extracts: "it is possible to elaborate federated queries over multiple end points to retrieve and link data from different RDF databases (DBs)"
- [intro] SPARQL is the designated language for extracting information from the RDF knowledge graph: "These graphs can be queried using the SPARQL Protocol and RDF Query Language (SPARQL), a language designed to query and extract information from RDF databases."
- [readme] Public GraphDB endpoint hosting the ENPKG graph requires no login: "The ENPKG graph is available at the following address https://enpkg.commons-lab.org/graphdb/. No need for login !"
- [readme] The SPARQL research interface location and the paper's example queries: "The SPARQL research interface can be reached at https://enpkg.commons-lab.org/graphdb/sparql. Make sure to check the [paper](https://doi.org/10.1021/acscentsci.3c00800) for examples of queries."
- [abstract] Semantic web technology enables repository-scale comparison of standardized data: "The use of semantic web technology enables comparison of processed and standardized data, information, and knowledge at the repository scale."
