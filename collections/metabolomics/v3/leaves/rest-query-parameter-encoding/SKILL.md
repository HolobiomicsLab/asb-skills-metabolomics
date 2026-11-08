---
name: rest-query-parameter-encoding
description: Use when you need annotations for exactly one PubChem record limited
  to a single (sub)heading (e.g. 'Solubility') via PUG-View, rather than the full
  annotation content for that record.
when_to_use_negative:
- You need annotations under multiple headings or for multiple record identifiers
  — a PUG-View request cannot take multiple input record identifiers and multi-heading
  retrieval requires a full annotation data request
- You need computed compound properties such as molecular weight — PUG-REST is the
  appropriate interface for individual computed properties
- The data of interest is specialized tabular content that comes from other backend
  services and is not accessible through PUG-View
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: REST-style web service that serves the heading-filtered single-record annotation
    request; its URL syntax (data/compound/{cid}/JSON?heading=...) is what this skill
    encodes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:b35395bc6008582f95133a69e0cb5f89058c9ec9c4652f6ee57a6b966822f7ac
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
n_claims: 8
---

# rest-query-parameter-encoding

## Summary

Constructs a single-record PUG-View request URL that encodes the three required pieces (annotation type 'data', record identifier, output format) and appends an optional 'heading' query parameter to restrict returned PubChem annotations to one (sub)heading.

## When to use

Use when you need annotations for exactly one PubChem record limited to a single (sub)heading (e.g. 'Solubility') via PUG-View, rather than the full annotation content for that record.

## When NOT to use

- You need annotations under multiple headings or for multiple record identifiers — a PUG-View request cannot take multiple input record identifiers and multi-heading retrieval requires a full annotation data request
- You need computed compound properties such as molecular weight — PUG-REST is the appropriate interface for individual computed properties
- The data of interest is specialized tabular content that comes from other backend services and is not accessible through PUG-View

## Inputs

- A single PubChem record identifier (e.g. compound CID 1983)
- Target (sub)heading name whose spaces are URL-encoded as '+' (e.g. 'Solubility', 'Experimental+Properties')
- Desired output format (JSON or XML)

## Outputs

- PUG-View JSON (or XML) annotation content restricted to the single requested (sub)heading, e.g. saved as solubility_cid1983.json
- An error message if the record has no annotations under the requested heading

## How to apply

Formulate a GET request to https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/{type}/{record}/{format} — for example annotation type 'data', Compound CID 1983, format JSON — then append the optional heading parameter with spaces URL-encoded as '+' (e.g. ?heading=Experimental+Properties or ?heading=Solubility). If unsure the heading exists, first issue an 'index' request that returns the record's Table of Contents without retrieving full data. Issue the HTTP GET within the 30-second request time limit, then parse the returned PUG-View JSON/XML content model and save it (e.g. as solubility_cid1983.json). The request returns an error message if the compound has no data under that heading, which signals that the heading or record must be re-checked.

## Related tools

- **PUG-View** (REST-style web service that serves the heading-filtered single-record annotation request; its URL syntax (data/compound/{cid}/JSON?heading=...) is what this skill encodes)

## Examples

```
curl 'https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON?heading=Solubility' > solubility_cid1983.json
```

## Evaluation signals

- Response must be valid JSON whose annotation content is restricted to the single requested (sub)heading
- Heading parameter spaces are URL-encoded as '+' in the request URL (e.g. Experimental+Properties)
- Request completes within the 30-second PUG-View time limit
- An error message (not annotation data) is returned when the compound has no data under that heading — treat this as a valid negative, not a failed request
- Optional pre-check: an 'index' request returns the record's Table of Contents confirming the heading exists before issuing the data request

## Limitations

- A PUG-View request cannot take multiple input record identifiers
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless full annotation data retrieval is requested
- If the compound has no data in the requested section, the request returns an error message
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View
- Requests are subject to a 30-second time limit

## Evidence

- [abstract] PUG-View is specialized for annotation retrieval: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [intro] Paper covers PUG-View request URL syntax: "The present paper will describe important aspects of PUG-View, including the scope of data accessible through PUG-View, the syntax for formulation a PUG-View request URL"
- [discussion] Single identifier per request: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Multi-heading retrieval requires full annotation request: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
