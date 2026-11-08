---
name: pubchem-interface-limitation-analysis
description: Use when designing a programmatic data-retrieval plan against PubChem
  and you need to decide whether a target data category (annotations, computed compound
  properties, bulk synonyms, ExternalTableName tables) is served by PUG-View, PUG-REST,
  both, or neither, and to identify fallback routes (FTP.
when_to_use_negative:
- Input is already a ready-made list of PubChem identifiers and you simply need to
  fetch data — use PUG-REST or PUG-View directly rather than re-deriving interface
  capabilities.
- The task is to formulate a specific PUG-View or PUG-REST request URL (query syntax),
  not to decide which interface can serve the data — that is a different skill.
- Target data are already confirmed to live in a single interface (e.g., computed
  compound properties known to be PUG-REST-only) and no further route-identification
  analysis is needed.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: PUG-View
  role: PubChem's REST-style interface specialized for accessing annotation data;
    one of the two routes being classified
- name: PUG-REST
  role: PubChem's most heavily used interface for structured data like computed compound
    properties and cross-references; the comparison route
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:ba29371bbe4a1d8e7ee91c66c1b927ccbcf97607715f51d8ea0653b94a68ba78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pubchem-interface-limitation-analysis

## Summary

A method for classifying which PubChem data categories are retrievable through PUG-View versus PUG-REST (or neither), based on each interface's design constraints such as single-record limitation, unstructured-vs-structured data handling, and backend service dependencies.

## When to use

Use when designing a programmatic data-retrieval plan against PubChem and you need to decide whether a target data category (annotations, computed compound properties, bulk synonyms, ExternalTableName tables) is served by PUG-View, PUG-REST, both, or neither, and to identify fallback routes (FTP, direct links) for data outside both interfaces.

## When NOT to use

- Input is already a ready-made list of PubChem identifiers and you simply need to fetch data — use PUG-REST or PUG-View directly rather than re-deriving interface capabilities.
- The task is to formulate a specific PUG-View or PUG-REST request URL (query syntax), not to decide which interface can serve the data — that is a different skill.
- Target data are already confirmed to live in a single interface (e.g., computed compound properties known to be PUG-REST-only) and no further route-identification analysis is needed.

## Inputs

- Article methods markdown of 'PUG-View: programmatic access to chemical annotations integrated in PubChem' (sections: Limitation Of PUG-View, Comparison Of PUG-View With PUG-REST, Conclusions)
- List of PubChem data categories of interest (e.g., textual annotations, computed compound properties, synonyms, InChI, patents, ExternalTableName tables, cross-references)

## Outputs

- TSV table (pug_view_vs_pug_rest_data_access.tsv) with columns: data_category, access_route (PUG-View / PUG-REST / either / neither), supporting_evidence_quote
- Explicit list of data categories requiring fallback routes (FTP site or other backend services)

## How to apply

Read the article's 'Limitation Of PUG-View', 'Comparison Of PUG-View With PUG-REST', and 'Conclusions' subsections of the Utility And Discussion section to extract every explicit accessibility statement. Classify each data category into one of four routes: PUG-View-only (e.g., textual annotations that PUG-REST cannot handle), PUG-REST-only (e.g., computed compound properties, multiple-record requests), either interface, or neither (e.g., bulk synonyms, InChI strings, patents, and tabular data flagged with 'ExternalTableName'/'ExternalTableNumRows' nodes that come from other backend services). For each classified row record a verbatim supporting evidence quote from the source text to ground the classification. Rationale: PUG-View was built specifically for annotation data that existing interfaces (including PUG-REST) were not designed to handle, so classifying data by interface capability prevents wasted request attempts and guides users toward the correct access route.

## Related tools

- **PUG-View** (PubChem's REST-style interface specialized for accessing annotation data; one of the two routes being classified)
- **PUG-REST** (PubChem's most heavily used interface for structured data like computed compound properties and cross-references; the comparison route)

## Examples

```
python scripts/classify_pubchem_routes.py --article article_methods.md --output pug_view_vs_pug_rest_data_access.tsv
```

## Evaluation signals

- Output file exists and parses as TSV with the three required columns (data_category, access_route, supporting_evidence_quote).
- Rows cover all four access-route classes: PUG-View, PUG-REST, either, and neither.
- At least one 'neither' row cites data marked with 'ExternalTableName'/'ExternalTableNumRows' nodes served by other backend services.
- Every row's evidence quote is verbatim from the Utility And Discussion section of the source article.
- PUG-View-only rows reference data types the article says existing interfaces (including PUG-REST) were not designed to handle.

## Limitations

- A PUG-View request cannot take multiple input record identifiers — multiple-record retrieval requires PUG-REST.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single PUG-View request unless full annotation data retrieval is requested.
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View.
- Bulk synonyms, InChI strings, and patent data are directed to the PubChem FTP site rather than either web-service interface.

## Evidence

- [discussion] PUG-View serves unstructured textual annotations and larger reports while PUG-REST serves readily structured data: "while PUG-REST is best for accessing specific individual computed compound properties"
- [discussion] some specialized tabular data come from other backend services and are not accessible through PUG-View: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
- [discussion] a PUG-View request cannot take multiple input record identifiers: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] annotation data under multiple headings cannot be retrieved with a single request: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
- [intro] existing web service interfaces in PubChem (including PUG-REST) were not designed to handle annotation data appropriately: "Because the existing web service interfaces in PubChem (including PUG-REST) were not designed to handle this type of data appropriately, PubChem developed a new REST-style interface, called PUG-View"
