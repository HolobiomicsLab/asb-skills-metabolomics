---
name: natural-product-analog-identification
description: Use when you need to prioritize potential structural analogs of a known
  bioactive compound (e.g., the anti-L. donovani triterpene isoiguesterin) among confident
  structural annotations of a natural product extract stored in an RDF knowledge graph.
when_to_use_negative:
- Input is a raw feature quantification table without structural annotations integrated
  into the RDF knowledge graph — run the ENPKG workflow (MZmine, ISDB/SIRIUS annotation,
  KG loading) first
- The goal is de novo structure elucidation rather than retrieving analogs of an already-known
  bioactive reference compound
- You need statistically validated activity predictions — the article explicitly reports
  no validation metric or ground-truth comparison for the analog identification
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_3172
tools:
- name: ENPKG
  role: Knowledge graph hosting the extract annotations, ChEMBL bioactivity data,
    NPClassifier taxonomy, and biosource provenance queried via SPARQL
  repo: https://github.com/enpkg/enpkg_full
- name: Sachem
  role: Structural-similarity/substructure search engine used inside SPARQL queries
    over the graph's 2D structures
- name: SPARQL
  role: Query language used, including federated queries over multiple RDF endpoints,
    to link similarity hits to activity and taxonomy annotations
- name: ChEMBL web resource client
  role: Access to ChEMBL anti-L. donovani activity annotations (ChEMBL367, DB v30)
    used as the reference compound set
- name: RDKit
  role: 2D structure handling supporting the Tanimoto similarity computation underlying
    the Sachem query
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:8bc3d11467dde9b5e1964027488d254c3d73d5791a8f22c465e0d8a5e344a0b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enpkg/enpkg_full
  tools:
  - ChEMBL web resource client
  - ENPKG
  - RDF
  - RDKit
  - SPARQL
  - Sachem
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# natural-product-analog-identification

## Summary

Identifies structural analogs of a bioactive reference compound among confident LC-MS/MS structural annotations by running Sachem structural-similarity SPARQL queries over the ENPKG knowledge graph and joining hits to ChEMBL activity annotations, NPClassifier taxonomy, and biosource provenance.

## When to use

Use when you need to prioritize potential structural analogs of a known bioactive compound (e.g., the anti-L. donovani triterpene isoiguesterin) among confident structural annotations of a natural product extract stored in an RDF knowledge graph.

## When NOT to use

- Input is a raw feature quantification table without structural annotations integrated into the RDF knowledge graph — run the ENPKG workflow (MZmine, ISDB/SIRIUS annotation, KG loading) first
- The goal is de novo structure elucidation rather than retrieving analogs of an already-known bioactive reference compound
- You need statistically validated activity predictions — the article explicitly reports no validation metric or ground-truth comparison for the analog identification

## Inputs

- Confident structural annotations from LC-MS/MS processing (ISDB + SIRIUS/CSI:FingerID) stored as RDF in the ENPKG knowledge graph
- Reference bioactive compound structure (e.g., isoiguesterin, a pristimerin-type triterpene)
- ChEMBL compounds with reported activity against the target (ChEMBL367 anti-L. donovani, NP-likeness score > −1, ChEMBL DB v30) in the KG
- Isolated compound accurate-mass data (HRESIMS m/z) for validation

## Outputs

- Ranked list of structural analogs with Tanimoto similarity ≥ 0.8 (e.g., 10 analogs among 99 confident annotations)
- CSV export (sachem_analog_query_results.csv) with structures, similarity relationships, ChEMBL activity metadata, NPClassifier taxonomy, and sample biosource provenance
- Validated top-candidate analog assignment (e.g., 11-β-hydroxypristimerin) with ppm mass-error check

## How to apply

Load reference compounds with reported bioactivity (e.g., ChEMBL367 anti-L. donovani compounds filtered to NP-likeness score > −1, ChEMBL DB v30) already integrated into the ENPKG knowledge graph. Run Sachem structural-similarity/substructure queries over the graph's 2D structures via the SPARQL research interface (https://enpkg.commons-lab.org/graphdb/sparql), filtering with a Tanimoto similarity threshold ≥ 0.8 against the reference compound (e.g., isoiguesterin). Use SPARQL joins — including federated queries over multiple RDF endpoints — to link similarity hits to ChEMBL activity annotations, NPClassifier chemical taxonomy, and biosource provenance stored as RDF. Verify top candidate analogs against the isolated compound's accurate-mass data (e.g., HRESIMS m/z 481.2950, calculated for C30H41O5, error 0.33 ppm) before export. Export the ranked analog list with structures, similarity relationships, activity metadata, and sample provenance to a CSV (sachem_analog_query_results.csv).

## Related tools

- **ENPKG** (Knowledge graph hosting the extract annotations, ChEMBL bioactivity data, NPClassifier taxonomy, and biosource provenance queried via SPARQL) — https://github.com/enpkg/enpkg_full
- **Sachem** (Structural-similarity/substructure search engine used inside SPARQL queries over the graph's 2D structures)
- **SPARQL** (Query language used, including federated queries over multiple RDF endpoints, to link similarity hits to activity and taxonomy annotations)
- **ChEMBL web resource client** (Access to ChEMBL anti-L. donovani activity annotations (ChEMBL367, DB v30) used as the reference compound set)
- **RDKit** (2D structure handling supporting the Tanimoto similarity computation underlying the Sachem query)

## Examples

```
curl -G "https://enpkg.commons-lab.org/graphdb/sparql" --data-urlencode "query=<Sachem structural-similarity SPARQL query vs ChEMBL367 anti-L. donovani compounds, Tanimoto >= 0.8>" --data-urlencode "format=csv" > sachem_analog_query_results.csv
```

## Evaluation signals

- Query returns 10 structures with Tanimoto similarity ≥ 0.8 against isoiguesterin among 99 confident structural annotations of the Pristimera indica extract
- Top candidate analog's calculated mass matches isolated compound HRESIMS m/z 481.2950 (C30H41O5) within reported ppm tolerance (0.33 ppm error)
- Exported CSV contains structures, similarity relationships, activity metadata, and sample provenance for each hit
- SPARQL joins correctly link similarity hits to ChEMBL anti-L. donovani annotations, NPClassifier classes, and biosource provenance
- Expert review confirms the output identifies 11-β-hydroxypristimerin as an active analog of reported anti-L. donovani compounds

## Limitations

- No validation metric or ground-truth comparison is given for the 'active analog of reported anti-L. donovani compounds' identification; validation relies on expert review
- No reported rate of false positives/false negatives from semantic merging of experimental data with public resources
- Confidence depends on agreement between ISDB and SIRIUS structural annotations used to strengthen annotation confidence before the similarity search
- Adoption requires SPARQL/semantic-web expertise; the authors note that great advances are still needed to democratize semantic web technologies in natural products research

## Evidence

- [results] Tanimoto similarity threshold ≥ 0.8 for structural analogs: "Tanimoto similarity superior or equal to 0.8 against isoiguesterin"
- [results] Sachem-based retrieval of 10 analogs among 99 confident annotations: [evidence span withheld — anchor status: not_found]
- [intro] Federated SPARQL queries link data across RDF endpoints: "it is possible to elaborate federated queries over multiple end points to retrieve and link data from different RDF databases (DBs)"
- [readme] SPARQL research interface for the ENPKG graph: "The SPARQL research interface can be reached at https://enpkg.commons-lab.org/graphdb/sparql. Make sure to check the [paper](https://doi.org/10.1021/acscentsci.3c00800) for examples of queries."
- [results] Combining ISDB and SIRIUS annotations raises confidence before analog search: [evidence span withheld — anchor status: not_found]
