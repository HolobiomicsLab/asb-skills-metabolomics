---
name: mass-spectrometry-feature-mass-matching
description: Use when an LC-MS2-annotated active extract needs its detected features
  (e.g. HRESIMS m/z 465.3002, 495.2754) matched to compounds in ChEMBL with reported
  activity against a target organism such as T. cruzi (CHEMBL368).
when_to_use_negative:
- The input is a raw feature table or unannotated MS2 spectra rather than a knowledge
  graph with structural annotations and ChEMBL cross-links.
- The organism of interest has no ChEMBL activity panel entry, so no bioactivity-annotated
  compounds exist to match masses against.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3362
tools:
- name: SPARQL
  role: Query language used to traverse the ENPKG knowledge graph and recover compounds
    matching observed feature masses via Wikidata and ChEMBL links.
  repo: https://enpkg.commons-lab.org/graphdb/
- name: ENPKG
  role: Knowledge-graph framework whose enriched annotations (enpkg:has_wd_id, ISDB/SIRIUS,
    NPClassifier, ChEMBL) provide the matchable compound records.
  repo: https://github.com/enpkg/enpkg_full
- name: RDF
  role: Graph data model in which experimental annotations and public knowledge are
    stored for mass/activity matching.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:aedbdfa10e78281db68706c18f2bb93e1dd367639818ccdef3debacf255b96f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ENPKG
  - RDF
  - SPARQL
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3362
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mass-spectrometry-feature-mass-matching

## Summary

Matches observed LC-MS2 feature masses from an active extract against annotated compounds with known bioactivity stored in the ENPKG knowledge graph (cross-referencing Wikidata and ChEMBL via SPARQL), to recover known anti-trypanosomatid constituents.

## When to use

Use when an LC-MS2-annotated active extract needs its detected features (e.g. HRESIMS m/z 465.3002, 495.2754) matched to compounds in ChEMBL with reported activity against a target organism such as T. cruzi (CHEMBL368).

## When NOT to use

- The input is a raw feature table or unannotated MS2 spectra rather than a knowledge graph with structural annotations and ChEMBL cross-links.
- The organism of interest has no ChEMBL activity panel entry, so no bioactivity-annotated compounds exist to match masses against.

## Inputs

- ENPKG knowledge graph (.ttl) imported into GraphDB with integrated ChEMBL bioactivity links
- SPARQL query traversing enpkg:has_wd_id, ISDB/SIRIUS annotations, NPClassifier taxonomy, and ChEMBL activity data
- Reported HRESIMS feature masses (e.g. 465.3002, 495.2754 in positive ionization mode) from the active extract
- Wikidata structure–organism pairs (LOTUS) for cross-organism annotation

## Outputs

- Table of matched compounds with name, Wikidata/ChEMBL ID, annotation type, and observed m/z
- CSV export (e.g. anti_tcruzi_showcase_results.csv) containing rows for each mass-matched known bioactive compound
- Count of distinct activity-annotated compounds and structure–organism pairs recovered from the extract

## How to apply

After feature detection and structural annotation (ISDB/SIRIUS) have populated the ENPKG knowledge graph, write a SPARQL query traversing enpkg:has_wd_id, ISDB/SIRIUS annotations, NPClassifier chemical taxonomy, and the integrated ChEMBL bioactivity links restricted to the biosource of interest (e.g. Pristimera indica, Q11075650). Run the query against the GraphDB SPARQL endpoint (https://enpkg.commons-lab.org/graphdb/sparql) and filter results to structural candidates whose annotated feature masses, measured in the relevant ionization mode, match reported HRESIMS values (e.g. 465.3002 for pristimerin and 495.2754 for zeylasteral in positive mode). Export matched rows (compound name, Wikidata/ChEMBL ID, annotation type, observed m/z) to a CSV for validation. The rationale is that mass matching between the sample's detected features and activity-annotated compounds in the KG rapidly recovers known bioactive constituents without re-dereplication.

## Related tools

- **SPARQL** (Query language used to traverse the ENPKG knowledge graph and recover compounds matching observed feature masses via Wikidata and ChEMBL links.) — https://enpkg.commons-lab.org/graphdb/
- **ENPKG** (Knowledge-graph framework whose enriched annotations (enpkg:has_wd_id, ISDB/SIRIUS, NPClassifier, ChEMBL) provide the matchable compound records.) — https://github.com/enpkg/enpkg_full
- **RDF** (Graph data model in which experimental annotations and public knowledge are stored for mass/activity matching.)

## Examples

```
curl -X POST -H 'Accept: application/sparql-results+json' --data-urlencode 'query=SELECT ?compound ?wd ?mz WHERE { ?feat enpkg:has_wd_id ?wd . ?wd <http://schema.org/sameAs> ?chembl . FILTER(?mz = 465.3002) }' https://enpkg.commons-lab.org/graphdb/sparql
```

## Evaluation signals

- Result table contains rows for both target compounds (e.g. pristimerin and zeylasteral) with m/z values matching the reported HRESIMS values (465.3002 and 495.2754).
- Every matched row carries a valid Wikidata ID and ChEMBL compound ID consistent with the annotation type recorded in the KG.
- Number of distinct activity-annotated compounds recovered from the extract matches the expected count from prior knowledge (e.g. 10 anti-T. cruzi compounds).
- Structure–organism pairs returned from Wikidata are consistent with the biosource constraint (e.g. Pristimera indica, Q11075650).
- Exported CSV columns include compound name, Wikidata/ChEMBL ID, annotation type, and observed m/z without missing values.

## Limitations

- No statistical metrics (e.g., FDR, p-values, effect sizes, AUROC) are reported for the anti-T. cruzi / anti-L. donovani compound annotations.
- No validation metric or ground-truth comparison is given for the 'active analog of reported anti-L. donovani compounds' identification.
- No reported rate of false positives/false negatives from semantic-merging of experimental data with public resources.

## Evidence

- [other] SPARQL queries over ENPKG with ChEMBL integration retrieved 10 distinct anti-T. cruzi compounds annotated in the active Melochia umbellata extract, all quinoline alkaloids reported in Waltheria indica, and 14 structure-organism pairs from Wikidata.: [evidence span withheld — anchor status: not_found]
- [other] Validation: result table rows must match pristimerin/zeylasteral masses: [evidence span withheld — anchor status: not_found]
- [other] SPARQL query traversing enpkg:has_wd_id, isdb/SIRIUS annotations, NPClassifier chemical taxonomy, and integrated ChEMBL bioactivity links restricted to the biosource Pristimera indica (Q11075650).: [evidence span withheld — anchor status: not_found]
