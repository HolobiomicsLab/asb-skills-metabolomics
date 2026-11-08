---
name: sparql-knowledge-graph-querying
description: Use when annotated LC-MS2 features in metabolomics samples must be linked
  to public knowledge (ChEMBL bioactivity, Wikidata/LOTUS structure–organism pairs,
  NPClassifier taxonomy) by traversing an RDF knowledge graph rather than flat feature
  tables.
when_to_use_negative:
- Input is still raw LC-MS2 data (.mzML/.mgf) that has not been processed by the ENPKG
  workflow and converted to RDF — run the ENPKG pipeline and graph builder first.
- The question requires RT-based feature alignment across batches — ENPKG is explicitly
  sample-centric precisely because RT-based alignment hinders comparison of samples
  from different batches or instrumental setups.
- A statistical benchmark (FDR, p-values, effect sizes) is required for the annotation
  calls — the paper reports no such validation metrics for the KG-derived compound
  identifications.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3314
tools:
- name: ENPKG
  role: Framework whose RDF knowledge graph (with ChEMBL, Wikidata/LOTUS, NPClassifier,
    and isdb/SIRIUS annotations integrated) is queried via SPARQL
  repo: https://github.com/enpkg/enpkg_workflow
- name: GraphDB
  role: Hosts the published ENPKG graph and its SPARQL research interface at https://enpkg.commons-lab.org/graphdb/sparql
  repo: https://graphdb.ontotext.com/documentation/
- name: Sachem
  role: Federated structural-similarity/substructure search over 2D structures in
    the graph, retrieving analogs such as 11-β-hydroxypristimerin at Tanimoto ≥ 0.8
- name: ChEMBL web resource client
  role: Provides ChEMBL bioactivity data (CHEMBL367/CHEMBL368 trypanosomatid activities)
    integrated into the graph and linked via SPARQL joins
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:3898f180ec8c8a3993badb258a4a4019929502a4f76a616c4d11f0656c6b7d9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ChEMBL web resource client
  - ENPKG
  - GraphDB
  - RDF
  - RDKit
  - SPARQL
  - Sachem
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# sparql-knowledge-graph-querying

## Summary

Query the ENPKG Experimental Natural Products Knowledge Graph (RDF, GraphDB) with SPARQL — including federated queries — to recover known bioactive compounds, structural analogs via Sachem similarity, and structure–organism pairs from integrated ChEMBL and Wikidata/LOTUS data.

## When to use

Use when annotated LC-MS2 features in metabolomics samples must be linked to public knowledge (ChEMBL bioactivity, Wikidata/LOTUS structure–organism pairs, NPClassifier taxonomy) by traversing an RDF knowledge graph rather than flat feature tables.

## When NOT to use

- Input is still raw LC-MS2 data (.mzML/.mgf) that has not been processed by the ENPKG workflow and converted to RDF — run the ENPKG pipeline and graph builder first.
- The question requires RT-based feature alignment across batches — ENPKG is explicitly sample-centric precisely because RT-based alignment hinders comparison of samples from different batches or instrumental setups.
- A statistical benchmark (FDR, p-values, effect sizes) is required for the annotation calls — the paper reports no such validation metrics for the KG-derived compound identifications.

## Inputs

- ENPKG knowledge graph (.ttl triples) loaded in GraphDB and published at https://enpkg.commons-lab.org/graphdb/
- SPARQL query specification (biosource Wikidata QID, ChEMBL target ID, NPClassifier class, similarity/mass thresholds)
- Reference accurate masses (HRESIMS m/z) of target known compounds

## Outputs

- Result table of annotated compounds with Wikidata/ChEMBL IDs, annotation type, biosource provenance, and observed m/z (e.g., anti_tcruzi_showcase_results.csv)
- Ranked structural-analog list with similarity relationships, activity metadata, and sample provenance (e.g., sachem_analog_query_results.csv)
- Structure–organism pairs retrieved from Wikidata via federated SPARQL

## How to apply

Open the ENPKG GraphDB endpoint (https://enpkg.commons-lab.org/graphdb/) and its SPARQL research interface (https://enpkg.commons-lab.org/graphdb/sparql), then write SPARQL queries traversing enpkg:has_wd_id, isdb/SIRIUS annotations, NPClassifier chemical taxonomy, and integrated ChEMBL activity links (e.g., T. cruzi CHEMBL368, L. donovani CHEMBL367). Use SPARQL joins to link Sachem structural-similarity hits (Tanimoto ≥ 0.8 against a query compound such as isoiguesterin) to activity annotations and biosource provenance; federated queries can pull Wikidata structure–organism pairs. Filter candidate rows by accurate mass — e.g., HRESIMS m/z 465.3002 (pristimerin) and 495.2754 (zeylasteral) in positive ionization mode, with mass errors within reported ppm tolerance (0.33 ppm for C30H41O5, m/z 481.2950). Export matched rows (compound name, Wikidata/ChEMBL ID, annotation type, observed m/z) to CSV and validate that expected known actives appear with correct masses.

## Related tools

- **ENPKG** (Framework whose RDF knowledge graph (with ChEMBL, Wikidata/LOTUS, NPClassifier, and isdb/SIRIUS annotations integrated) is queried via SPARQL) — https://github.com/enpkg/enpkg_workflow
- **GraphDB** (Hosts the published ENPKG graph and its SPARQL research interface at https://enpkg.commons-lab.org/graphdb/sparql) — https://graphdb.ontotext.com/documentation/
- **Sachem** (Federated structural-similarity/substructure search over 2D structures in the graph, retrieving analogs such as 11-β-hydroxypristimerin at Tanimoto ≥ 0.8)
- **ChEMBL web resource client** (Provides ChEMBL bioactivity data (CHEMBL367/CHEMBL368 trypanosomatid activities) integrated into the graph and linked via SPARQL joins)

## Examples

```
curl -H "Accept: application/sparql-results+json" --data-urlencode "query=SELECT ?compound ?wd_id ?mz WHERE { ?f a enpkg:LCMSFeature ; enpkg:has_wd_id ?wd_id ; enpkg:has_annotation ?compound ; enpkg:observed_mass ?mz . }" https://enpkg.commons-lab.org/graphdb/sparql
```

## Evaluation signals

- Result table contains rows for both pristimerin and zeylasteral with m/z values matching 465.3002 and 495.2754 in positive ionization mode
- Sachem similarity query returns 10 structures with Tanimoto ≥ 0.8 against isoiguesterin among 99 confident structural annotations, and top analog matches m/z 481.2950 within 0.33 ppm
- SPARQL query recovers 10 distinct anti-T. cruzi compounds annotated in the active Melochia umbellata extract plus 14 structure–organism pairs from Wikidata
- Korean Pharmacopeia (MSV00008616/MSV000093464) triples are retrievable at the ENPKG SPARQL endpoint under enpkg:has_lcms_feature_list_a137fd4a263d3587d35f61a526932c09 (PI mode)
- Exported CSV rows carry valid Wikidata IDs (enpkg:has_wd_id) and ChEMBL IDs with consistent annotation types and biosource QIDs

## Limitations

- No statistical metrics (FDR, p-values, effect sizes, AUROC) are reported for the anti-T. cruzi / anti-L. donovani compound annotations; validation is by expert review of masses and analog relationships.
- No reported rate of false positives/false negatives from semantic merging of experimental data with public resources (LOTUS/Wikidata/ChEMBL).
- Great advances are still needed to democratize semantic web technologies in natural products research; writing SPARQL queries and KG schemas is a barrier.

## Evidence

- [results] SPARQL queries over ENPKG with ChEMBL integration retrieved 10 distinct anti-T. cruzi compounds annotated in the active Melochia umbellata extract: [evidence span withheld — anchor status: not_found]
- [intro] Federated queries over multiple endpoints: "it is possible to elaborate federated queries over multiple end points to retrieve and link data from different RDF databases (DBs)"
- [results] Sachem similarity threshold: "Tanimoto similarity superior or equal to 0.8 against isoiguesterin"
- [readme] GraphDB endpoint in README: "The ENPKG graph is available at the following address https://enpkg.commons-lab.org/graphdb/. No need for login !"
- [readme] SPARQL interface in README: "The SPARQL research interface can be reached at https://enpkg.commons-lab.org/graphdb/sparql. Make sure to check the [paper](https://doi.org/10.1021/acscentsci.3c00800) for examples of queries."
