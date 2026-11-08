---
name: restful-url-construction
description: Use when you need to programmatically fetch full PubChem annotation/summary
  content for a specific CID, SID, or AID record and must formulate a valid PUG-View
  request URL.
when_to_use_negative:
- You need specific individual computed compound properties rather than integrated
  annotation content — PUG-REST is the better interface for that purpose
- You want to batch-retrieve annotations for multiple record identifiers in a single
  request — PUG-View cannot take multiple input record identifiers
- You need specialized tabular data that comes from other backend services and is
  not currently accessible through PUG-View
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: REST-style web service interface whose request-URL syntax is constructed to
    retrieve full PubChem annotation data for a single record
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:a300060af644495ec195b62a9122698e259c7d1c3e7f36ab29a291afa5277def
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# restful-url-construction

## Summary

Constructs PUG-View REST-style request URLs that embed database, record identifier, and output format to programmatically retrieve full chemical annotation data from PubChem for a single compound, substance, or assay record.

## When to use

Use when you need to programmatically fetch full PubChem annotation/summary content for a specific CID, SID, or AID record and must formulate a valid PUG-View request URL.

## When NOT to use

- You need specific individual computed compound properties rather than integrated annotation content — PUG-REST is the better interface for that purpose
- You want to batch-retrieve annotations for multiple record identifiers in a single request — PUG-View cannot take multiple input record identifiers
- You need specialized tabular data that comes from other backend services and is not currently accessible through PUG-View

## Inputs

- PubChem record identifiers (single CID, SID, or AID, e.g. CID 1983)
- Target database specifier (compound, substance, or assay)
- Desired output format (JSON)

## Outputs

- Per-record JSON annotation files (e.g. annotations_cid1983.json, annotations_sid46506142.json, annotations_aid1259416.json) containing the 'Record' node with full annotation data

## How to apply

Build a PUG-View URL with the three-part syntax: the annotation type ('data'), the record specification (a single compound, substance, or assay identifier such as CID 1983, SID 46506142, or AID 1259416), and the desired output format ('JSON'). Issue the HTTP GET against https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/{compound|substance|assay}/{identifier}/JSON, one record per request, because PUG-View cannot take multiple input record identifiers in a single URL. Save each JSON response (e.g., annotations_cid1983.json) and validate that it parses as valid JSON and contains the 'Record' node for the requested identifier. If annotations under multiple headings are needed, either issue separate requests per heading or request the full annotation data, since partial multi-heading retrieval in one request is not supported.

## Related tools

- **PUG-View** (REST-style web service interface whose request-URL syntax is constructed to retrieve full PubChem annotation data for a single record)

## Examples

```
curl -o annotations_cid1983.json https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON
```

## Evaluation signals

- Each saved response parses as valid JSON
- Each returned JSON contains the 'Record' node for the requested identifier
- Annotation content in the JSON matches what is served on the corresponding PubChem Summary/Record web page
- Responses returning an error message (e.g., unknown record) are reported as failures rather than saved as results
- Each request URL contains exactly one record identifier — no multi-identifier URLs are attempted

## Limitations

- A PUG-View request cannot take multiple input record identifiers
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View

## Evidence

- [abstract] PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem: "PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem"
- [abstract] the scope of data accessible through PUG-View, the syntax for formulating a PUG-View request URL: [evidence span withheld — anchor status: not_found]
- [discussion] a PUG-View request cannot take multiple input record identifiers: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
