---
name: pubchem-annotation-data-classification
description: Decision support for choosing between PUG-View and PUG-REST (or FTP/other
  backends) when retrieving PubChem annotation data, computed properties, bulk synonyms,
  or ExternalTableName tabular content.
when_to_use_negative:
- The task is executing a PUG-REST or PUG-View request itself (e.g., fetching a compound's
  computed property) rather than deciding which interface can serve a data category
- The input is not PubChem-related; e.g., data access routing for E-Utilities, PUG-SOAP,
  or generic REST APIs outside PubChem
- A structured data table with explicit per-record interface metadata already exists,
  making manual classification of access routes redundant
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: PUG-View
  role: REST-style PubChem interface specialized for accessing annotation data; the
    route assigned to unstructured textual annotation content in the classification
- name: PUG-REST
  role: PubChem interface for readily structured data (computed compound properties,
    activity data, cross-references) and the only route supporting multiple input
    record identifiers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:0a574a8d4c7925ccd2c274e266414f1c7cdd5f010cff467fdede70c5064bd93e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pubchem-annotation-data-classification

## Summary

A curation skill that classifies PubChem data categories into access routes (PUG-View, PUG-REST, either, neither) based on the accessibility statements in the PUG-View paper, so agents route annotation requests to the correct interface or fallback service.

## When to use

Decision support for choosing between PUG-View and PUG-REST (or FTP/other backends) when retrieving PubChem annotation data, computed properties, bulk synonyms, or ExternalTableName tabular content.

## When NOT to use

- The task is executing a PUG-REST or PUG-View request itself (e.g., fetching a compound's computed property) rather than deciding which interface can serve a data category
- The input is not PubChem-related; e.g., data access routing for E-Utilities, PUG-SOAP, or generic REST APIs outside PubChem
- A structured data table with explicit per-record interface metadata already exists, making manual classification of access routes redundant

## Inputs

- Article methods/discussion markdown of the PUG-View paper (DOI 10.1186/s13321-019-0375-2), specifically the Utility And Discussion subsections
- List of PubChem data categories to classify (e.g., textual annotations, computed compound properties, cross-references, bulk synonyms, InChI strings, patents, ExternalTableName tables)

## Outputs

- pug_view_vs_pug_rest_data_access.tsv (tab-separated, UTF-8) with rows: data category, access route (PUG-View | PUG-REST | either | neither), verbatim supporting evidence quote

## How to apply

Load the PUG-View article's Utility And Discussion subsections ('Limitation Of Pug-View', 'Comparison Of Pug-View With Pug-Rest', 'Conclusions') and extract every explicit accessibility statement. Assign each data category to one of four routes: PUG-View (unstructured textual annotation reports and summary information), PUG-REST (readily structured data such as computed compound properties, activity data, and cross-references; the only route supporting multiple input record identifiers), either, or neither (e.g., bulk synonyms/InChI strings/patents directed to the PubChem FTP site, and tables flagged with 'ExternalTableName'/'ExternalTableNumRows' nodes served by other backend services). Record each classified row with a verbatim supporting quote, then validate that all four route classes are represented, including at least one 'neither' row citing ExternalTableName-marked tabular data. The rationale is that existing interfaces including PUG-REST were not designed to handle annotation data, which motivated PUG-View, so route choice depends on data structure rather than convenience.

## Related tools

- **PUG-View** (REST-style PubChem interface specialized for accessing annotation data; the route assigned to unstructured textual annotation content in the classification)
- **PUG-REST** (PubChem interface for readily structured data (computed compound properties, activity data, cross-references) and the only route supporting multiple input record identifiers)

## Evaluation signals

- Output file exists and parses as TSV with the three required columns (data category, access route, evidence quote)
- All four access-route classes (PUG-View, PUG-REST, either, neither) are represented in the classified rows
- At least one 'neither' row cites ExternalTableName-marked tabular data as served by other backend services
- Every row's evidence is a verbatim quote traceable to the Utility And Discussion section
- Multiple-record requests are classified as PUG-REST-only, consistent with 'a PUG-View request cannot take multiple input record identifiers'

## Limitations

- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View
- A PUG-View request cannot take multiple input record identifiers
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested

## Evidence

- [abstract] PUG-View is a REST-style annotation interface: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [intro] Existing interfaces unsuitable for annotations: "Because the existing web service interfaces in PubChem (including PUG-REST) were not designed to handle this type of data appropriately, PubChem developed a new REST-style interface"
- [discussion] Tabular data outside PUG-View: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
- [discussion] PUG-REST for computed compound properties: "while PUG-REST is best for accessing specific individual computed compound properties"
- [discussion] Single-record restriction of PUG-View: "a PUG-View request cannot take multiple input record identifiers"
