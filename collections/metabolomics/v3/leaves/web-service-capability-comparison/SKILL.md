---
name: web-service-capability-comparison
description: Use when a task asks which kinds of repository data (e.g., computed properties,
  annotations, bulk synonyms, tabular data) are served by each of several programmatic
  interfaces, and which fall outside all of them.
when_to_use_negative:
- The task only requires retrieving annotation data from one interface (e.g., a PUG-View
  request for a single record) rather than systematically comparing interface capabilities.
- The input text contains no explicit accessibility or limitation statements distinguishing
  the interfaces — classification rows would be invented rather than grounded in verbatim
  quotes.
- The goal is bulk structured data retrieval (e.g., computed properties, synonyms)
  where PUG-REST or the PubChem FTP site is the known correct route and no interface
  comparison is needed.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: The REST-style PubChem interface specialized for accessing annotation data;
    one side of the capability comparison and the route serving larger unstructured
    textual annotation reports.
  repo: https://pubchem.ncbi.nlm.nih.gov
- name: PUG-REST
  role: The heavily used PubChem interface best for accessing specific individual
    computed compound properties; contrasted against PUG-View to identify PUG-REST-only
    capabilities such as multiple-record requests.
  repo: https://pubchem.ncbi.nlm.nih.gov
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:e958f82b863a5631eaf85dbe238643f45035cda5f198e973ee4e3056b14116e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# web-service-capability-comparison

## Summary

A method for systematically comparing which data categories are accessible through which programmatic web service interfaces of a data repository — here, PubChem's PUG-View (unstructured textual annotation reports) versus PUG-REST (structured data such as computed compound properties and cross-references) — and flagging data reachable through neither interface.

## When to use

Use when a task asks which kinds of repository data (e.g., computed properties, annotations, bulk synonyms, tabular data) are served by each of several programmatic interfaces, and which fall outside all of them.

## When NOT to use

- The task only requires retrieving annotation data from one interface (e.g., a PUG-View request for a single record) rather than systematically comparing interface capabilities.
- The input text contains no explicit accessibility or limitation statements distinguishing the interfaces — classification rows would be invented rather than grounded in verbatim quotes.
- The goal is bulk structured data retrieval (e.g., computed properties, synonyms) where PUG-REST or the PubChem FTP site is the known correct route and no interface comparison is needed.

## Inputs

- Article methods markdown containing the Utility and Discussion section with 'Limitation Of Pug-View', 'Comparison Of Pug-View With Pug-Rest', and 'Conclusions' subsections
- List of data categories mentioned in the comparison (e.g., computed compound properties, activity data, cross-references, annotations, bulk synonyms, InChI strings, patents, ExternalTableName-marked tables)

## Outputs

- pug_view_vs_pug_rest_data_access.tsv — tab-separated, UTF-8 file with three required columns mapping each data category to its access route (PUG-View, PUG-REST, either, neither) plus a verbatim supporting evidence quote per row

## How to apply

Locate the comparison and limitation subsections of the article's Utility and Discussion (e.g., 'Limitation Of Pug-View', 'Comparison Of Pug-View With Pug-Rest', 'Conclusions') and extract every explicit accessibility statement verbatim. Classify each data category into one of four access routes: PUG-View-only, PUG-REST-only, either interface, or neither. The paper's own distinctions anchor the classification: PUG-REST handles readily structured data such as computed compound properties, activity data, and cross-references and is the only interface supporting multiple input record identifiers, while PUG-View serves larger, unstructured textual annotation reports. Categories the paper marks as outside both interfaces — bulk synonyms/InChI strings/patents directed to the PubChem FTP site, and specialized (especially tabular) data flagged with 'ExternalTableName'/'ExternalTableNumRows' nodes served by other backend services — must be captured as 'neither' rows. Write each classified row to a TSV with a verbatim supporting evidence quote, then validate the file parses with the required columns and covers all four route classes.

## Related tools

- **PUG-View** (The REST-style PubChem interface specialized for accessing annotation data; one side of the capability comparison and the route serving larger unstructured textual annotation reports.) — https://pubchem.ncbi.nlm.nih.gov
- **PUG-REST** (The heavily used PubChem interface best for accessing specific individual computed compound properties; contrasted against PUG-View to identify PUG-REST-only capabilities such as multiple-record requests.) — https://pubchem.ncbi.nlm.nih.gov

## Evaluation signals

- The output TSV parses correctly and contains the three required columns (data category, access route, verbatim evidence quote).
- All four access-route classes (PUG-View, PUG-REST, either, neither) are represented by at least one classified row.
- At least one 'neither' row cites ExternalTableName-marked tabular data as being served by other backend services rather than PUG-View.
- Every row's evidence quote is verbatim text drawn from the Utility and Discussion section of the article.
- Multiple-record requests are classified as PUG-REST-only, consistent with the article's statement that a PUG-View request cannot take multiple input record identifiers.

## Limitations

- PUG-View requests cannot take multiple input record identifiers, so multi-record comparison tasks must go through PUG-REST instead.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single PUG-View request unless full annotation data retrieval is requested.
- Some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View; related data such as bulk synonyms, InChI strings, and patents must be obtained from the PubChem FTP site instead of either web interface.

## Evidence

- [discussion] PUG-View serves unstructured textual annotation reports: "PUG-View allows one to programmatically access this annotation content in PubChem and is also used as a backend service"
- [discussion] PUG-REST serves structured data such as computed compound properties: "while PUG-REST is best for accessing specific individual computed compound properties"
- [discussion] Some specialized tabular data are outside PUG-View and come from other backend services: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
- [discussion] Multiple-record requests are a PUG-REST-only capability: "a PUG-View request cannot take multiple input record identifiers"
- [intro] Existing interfaces including PUG-REST could not handle annotation data, motivating PUG-View: "Because the existing web service interfaces in PubChem (including PUG-REST) were not designed to handle this type of data appropriately, PubChem developed a new REST-style interface, called PUG-View"
- [abstract] PUG-View is specialized for annotation data access: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
