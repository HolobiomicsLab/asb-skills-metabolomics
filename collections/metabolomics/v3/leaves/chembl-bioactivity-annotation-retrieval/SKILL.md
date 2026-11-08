---
name: chembl-bioactivity-annotation-retrieval
description: Use when confident structural annotations from LC-MS/MS metabolomics
  (e.g., ISDB/SIRIUS annotations in an RDF knowledge graph) must be connected to reported
  bioactivities in ChEMBL to rank analogs of known actives.
when_to_use_negative:
- The features or compounds under study have no confident structural annotations in
  the graph and no candidate structures — the join needs annotated structures to link
  to ChEMBL activities
- The target organism or activity of interest has no pre-integrated ChEMBL compound
  set in the knowledge graph and the SPARQL endpoint cannot be extended
- A de novo structure elucidation (not retrieval of analogs of already-reported actives)
  is the goal
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ENPKG
  role: Knowledge graph integrating ChEMBL bioactivity annotations, structural annotations,
    NPClassifier taxonomy, and sample provenance as RDF
  repo: https://github.com/enpkg/enpkg_workflow
- name: SPARQL
  role: Query language used to join similarity hits to ChEMBL activity annotations,
    NPClassifier taxonomy, and biosource provenance in the KG
  repo: https://enpkg.commons-lab.org/graphdb/
- name: Sachem
  role: Structural-similarity/substructure search over the graph's 2D structures to
    retrieve analogs (Tanimoto ≥ 0.8) of the compound of interest
- name: ChEMBL web resource client
  role: Accessing ChEMBL activity data for target compounds (e.g., CHEMBL367 anti-L.
    donovani set)
- name: RDF
  role: Standard graph model in which the experimental data and public ChEMBL knowledge
    are integrated for semantic querying
- name: RDKit
  role: Chemoinformatics support for structure handling (e.g., Tanimoto similarity
    computation) in the analog-retrieval workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:d8118130e4b93c138926c52f9d5f2d6c4b28cbd68bfeece11ac1613e62736656
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_workflow
  tools:
  - ChEMBL web resource client
  - ENPKG
  - RDF
  - RDKit
  - SPARQL
  - Sachem
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chembl-bioactivity-annotation-retrieval

## Summary

Retrieves and joins reported bioactivity annotations from ChEMBL (e.g., CHEMBL367 anti-Leishmania donovani compounds) into the ENPKG knowledge graph via SPARQL, linking experimentally annotated sample features to known actives for analog prioritization.

## When to use

Use when confident structural annotations from LC-MS/MS metabolomics (e.g., ISDB/SIRIUS annotations in an RDF knowledge graph) must be connected to reported bioactivities in ChEMBL to rank analogs of known actives.

## When NOT to use

- The features or compounds under study have no confident structural annotations in the graph and no candidate structures — the join needs annotated structures to link to ChEMBL activities
- The target organism or activity of interest has no pre-integrated ChEMBL compound set in the knowledge graph and the SPARQL endpoint cannot be extended
- A de novo structure elucidation (not retrieval of analogs of already-reported actives) is the goal

## Inputs

- ENPKG knowledge graph (RDF) with confident structural annotations of sample features (e.g., 99 annotations for a P. indica extract)
- ChEMBL compounds with reported activity against the target organism (CHEMBL367, anti-L. donovani, ChEMBL DB v30, NP-likeness score > −1)
- Sachem structural-similarity index over the graph's 2D structures
- Query compound structure (e.g., isoiguesterin) and/or accurate-mass reference data (HRESIMS m/z, formula, ppm tolerance)

## Outputs

- Ranked analog list with structures, Tanimoto similarity relationships, ChEMBL activity metadata, NPClassifier taxonomy, and sample provenance (e.g., sachem_analog_query_results.csv)
- Validated candidate annotations with mass-error check against experimental HRESIMS data (e.g., 0.33 ppm for C30H41O5)

## How to apply

Load the pre-integrated set of ChEMBL compounds with reported activity against the target organism — e.g., CHEMBL367 compounds active against Leishmania donovani, filtered to NP-likeness score > −1 from ChEMBL DB v30 — already stored as RDF in the ENPKG knowledge graph. Query the KG through the SPARQL research interface (https://enpkg.commons-lab.org/graphdb/sparql), optionally using Sachem structural-similarity or substructure predicates to find analogs (e.g., Tanimoto similarity ≥ 0.8 against the query compound) among confident sample annotations. Use SPARQL joins (including federated queries over multiple RDF endpoints) to link similarity hits to ChEMBL activity annotations, NPClassifier chemical taxonomy, and biosource provenance. Validate top candidates against experimental accurate-mass data, requiring the mass error to fall within the reported ppm tolerance (e.g., HRESIMS m/z 481.2950, calculated for C30H41O5, error 0.33 ppm). Export the ranked list with structures, similarity relationships, activity metadata, and sample provenance (e.g., to sachem_analog_query_results.csv) for expert review.

## Related tools

- **ENPKG** (Knowledge graph integrating ChEMBL bioactivity annotations, structural annotations, NPClassifier taxonomy, and sample provenance as RDF) — https://github.com/enpkg/enpkg_workflow
- **SPARQL** (Query language used to join similarity hits to ChEMBL activity annotations, NPClassifier taxonomy, and biosource provenance in the KG) — https://enpkg.commons-lab.org/graphdb/
- **Sachem** (Structural-similarity/substructure search over the graph's 2D structures to retrieve analogs (Tanimoto ≥ 0.8) of the compound of interest)
- **ChEMBL web resource client** (Accessing ChEMBL activity data for target compounds (e.g., CHEMBL367 anti-L. donovani set))
- **RDF** (Standard graph model in which the experimental data and public ChEMBL knowledge are integrated for semantic querying)
- **RDKit** (Chemoinformatics support for structure handling (e.g., Tanimoto similarity computation) in the analog-retrieval workflow)

## Examples

```
from chembl_webresource_client.new_client import new_client; acts = new_client.activity.filter(target_chembl_id='CHEMBL367')
```

## Evaluation signals

- The output recovers analogs at Tanimoto similarity ≥ 0.8 against the query compound among the sample's confident annotations (e.g., 10 of 99 annotations in the P. indica case)
- The top candidate is confirmed as an analog of a reported anti-L. donovani compound (e.g., 11-β-hydroxypristimerin vs isoiguesterin) upon expert review
- Accurate-mass verification: the candidate's calculated formula matches experimental HRESIMS within the reported ppm tolerance (0.33 ppm for m/z 481.2950, C30H41O5)
- Each retrieved analog row carries joined ChEMBL activity metadata, NPClassifier taxonomic class, and biosource provenance from the KG
- The query runs successfully against the public SPARQL research interface (https://enpkg.commons-lab.org/graphdb/sparql) without authentication

## Limitations

- No statistical validation metric or ground-truth comparison is reported for the active-analog identification; final validation rests on expert review and mass-error agreement
- The approach depends on pre-integrated public knowledge (ChEMBL compound sets with NP-likeness filtering at a specific DB version, v30) in the KG; new or non-integrated resources require custom ingestion
- The paper notes that no reported rate of false positives/false negatives exists for the semantic merging of experimental data with public resources

## Evidence

- [results] Tanimoto similarity threshold for analog retrieval: "Tanimoto similarity superior or equal to 0.8 against isoiguesterin"
- [intro] Federated SPARQL queries over multiple RDF endpoints link hits to activity annotations: "it is possible to elaborate federated queries over multiple end points to retrieve and link data from different RDF databases (DBs)"
- [readme] Public SPARQL research interface for querying the ENPKG graph: "The SPARQL research interface can be reached at https://enpkg.commons-lab.org/graphdb/sparql. Make sure to check the [paper](https://doi.org/10.1021/acscentsci.3c00800) for examples of queries."
- [intro] Use-case goal: identify analogs of reported anti-L. donovani compounds via the KG: "show how the KG structure can help answer research questions and lead to the identification of new anti-T. cruzi and anti-L. donovani compounds."
- [results] ChEMBL similarity hits joined to NPClassifier taxonomy and biosource provenance: "an in silico database of natural products (ISDB-LOTUS)"
