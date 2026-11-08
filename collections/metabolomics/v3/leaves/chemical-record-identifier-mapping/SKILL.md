---
name: chemical-record-identifier-mapping
description: Use when a task needs the complete annotation/summary content for one
  specific PubChem record (e.g. Compound Summary for a CID) via programmatic access,
  and the record can be named by a single CID, SID, or AID.
when_to_use_negative:
- The task requires retrieving annotations for many records in one call — a PUG-View
  request cannot take multiple input record identifiers.
- The task needs specific individual computed compound properties — use PUG-REST,
  which is best suited for that.
- The task needs specialized (especially tabular) data that come from other backend
  services and are not accessible through PUG-View.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0218
tools:
- name: PUG-View
  role: REST-style web service interface through which the three-part annotation request
    URL (data / record identifier / output format) is issued via HTTP GET to fetch
    full-record PubChem annotations.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:95033da9887062611b5a81f2da42192f631ddaf093fd8ab6229dc3d007d2ba01
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chemical-record-identifier-mapping

## Summary

Retrieves full-record annotation content for a single PubChem record (compound CID, substance SID, or assay AID) by constructing a PUG-View REST request URL that embeds the database, the record identifier, and the output format, then saving the returned JSON.

## When to use

Use when a task needs the complete annotation/summary content for one specific PubChem record (e.g. Compound Summary for a CID) via programmatic access, and the record can be named by a single CID, SID, or AID.

## When NOT to use

- The task requires retrieving annotations for many records in one call — a PUG-View request cannot take multiple input record identifiers.
- The task needs specific individual computed compound properties — use PUG-REST, which is best suited for that.
- The task needs specialized (especially tabular) data that come from other backend services and are not accessible through PUG-View.

## Inputs

- single PubChem record identifier (CID for compound, SID for substance, AID for assay)
- target database domain (compound | substance | assay)
- desired output format (JSON)

## Outputs

- full-record annotation JSON file (e.g. annotations_cid1983.json) containing the 'Record' node for the requested identifier
- failure report for requests that return an error message (e.g. unknown record)

## How to apply

Construct a three-part PUG-View URL: the annotation type ('data'), the record specification (one identifier within compound, substance, or assay), and the desired output format ('JSON'), e.g. https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON for the Compound Summary annotations of CID 1983. Issue one HTTP GET request per record because a PUG-View request cannot take multiple input record identifiers. Requesting the full annotation data retrieval is the way to obtain content spanning multiple headings in one call, since per-heading subsets are restricted to single headings per request. Save each response to disk (e.g. annotations_cid1983.json, annotations_sid46506142.json, annotations_aid1259416.json) and validate that it parses as JSON and contains the 'Record' node for the requested identifier.

## Related tools

- **PUG-View** (REST-style web service interface through which the three-part annotation request URL (data / record identifier / output format) is issued via HTTP GET to fetch full-record PubChem annotations.)

## Examples

```
curl https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON > annotations_cid1983.json
```

## Evaluation signals

- Each saved file parses as valid JSON.
- Each response contains the 'Record' node for the requested identifier (CID, SID, or AID).
- Saved annotation content matches what is served for the corresponding Summary/Record web page.
- Exactly one record identifier is embedded per request URL (no multi-identifier requests).
- Responses returning an error message (e.g. unknown record) are flagged as failures rather than silently accepted.

## Limitations

- A PUG-View request cannot take multiple input record identifiers, so bulk retrieval requires one request per record.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested.
- Some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View.

## Evidence

- [abstract] One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem.: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [abstract] the syntax for formulating a PUG-View request URL: "The present paper describes various aspects of PUG-View, including the scope of data accessible through PUG-View, the syntax for formulating a PUG-View request URL"
- [discussion] a PUG-View request cannot take multiple input record identifiers: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
