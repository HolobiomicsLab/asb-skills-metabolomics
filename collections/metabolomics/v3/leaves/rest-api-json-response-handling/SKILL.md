---
name: rest-api-json-response-handling
description: Use when a downstream analysis consumes data retrieved from a REST-style
  endpoint that returns JSON documents — e.g. chemical annotation records from PUG-View,
  whose response follows the PUG-View annotation content model (the same model as
  the ASNT/ASNB formats).
when_to_use_negative:
- When multiple record identifiers must be queried in a single call, since a PUG-View
  request cannot take multiple input record identifiers.
- When the desired data are specialized tabular data served from other backend services
  that are not currently accessible through PUG-View.
- When you need to retrieve annotation data under multiple headings for a given compound
  without requesting the full annotation retrieval, as a single request cannot do
  that.
tools:
- name: PUG-View
  role: REST-style web service interface specialized for accessing annotation data
    contained in PubChem; serves the JSON response payload consumed by this skill.
- name: PUG-REST
  role: Alternative PubChem REST interface best for accessing specific individual
    computed compound properties; PUG-View is preferred when annotation content rather
    than computed properties is needed.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:54f423a9fc1d4c51f794bb95ba75cfd12b8e53e2f26375700d762e9792e1f50e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rest-api-json-response-handling

## Summary

Handles parsing and validating JSON response payloads returned by REST-style web service interfaces, exemplified here by PubChem's PUG-View service, ensuring programmatic access to structured annotation data.

## When to use

Use when a downstream analysis consumes data retrieved from a REST-style endpoint that returns JSON documents — e.g. chemical annotation records from PUG-View, whose response follows the PUG-View annotation content model (the same model as the ASNT/ASNB formats).

## When NOT to use

- When multiple record identifiers must be queried in a single call, since a PUG-View request cannot take multiple input record identifiers.
- When the desired data are specialized tabular data served from other backend services that are not currently accessible through PUG-View.
- When you need to retrieve annotation data under multiple headings for a given compound without requesting the full annotation retrieval, as a single request cannot do that.

## Inputs

- HTTP GET request URL for the REST-style endpoint (e.g. https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/887/JSON)
- Optional query parameters such as toc=LCSS+TOC for reorganizing annotations
- Single record identifier (e.g. CID 887)

## Outputs

- Valid JSON response payload following the PUG-View annotation content model
- Output file such as lcss_cid887.json containing the requested annotation data

## How to apply

Issue an HTTP GET request to the REST-style endpoint (e.g. https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/887/JSON), ensuring only a single record identifier is encoded in the URL since PUG-View does not accept multiple identifiers. Optional parameters such as toc=LCSS+TOC (space encoded as the plus character) reorganize the record annotations according to a specific table of contents instead of the default one. Parse the returned JSON document, verifying that it contains the expected annotation sections for the requested record rather than the full summary content. Respect the service's usage policies — no more than 5 requests per second and a 30-second per-request time limit. Write the parsed JSON payload to an output file (e.g. lcss_cid887.json) for downstream use. Because PUG-View returns annotation content rather than the tabular computed properties served by PUG-REST, this skill focuses on handling the hierarchical annotation content model rather than flat property tables.

## Related tools

- **PUG-View** (REST-style web service interface specialized for accessing annotation data contained in PubChem; serves the JSON response payload consumed by this skill.)
- **PUG-REST** (Alternative PubChem REST interface best for accessing specific individual computed compound properties; PUG-View is preferred when annotation content rather than computed properties is needed.)

## Examples

```
curl -o lcss_cid887.json 'https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/887/JSON?toc=LCSS+TOC'
```

## Evaluation signals

- The output file exists and is valid JSON.
- The parsed document contains the LCSS-organized health and safety annotation data for the requested record (e.g. CID 887) rather than the full Compound Summary content.
- The response follows the PUG-View annotation content model (the same model as the ASNT/ASNB formats).
- Request rate stayed within the policy limit of no more than 5 requests per second and the 30-second per-request time limit.
- Only a single record identifier was encoded in the request URL.

## Limitations

- A PUG-View request cannot take multiple input record identifiers.
- Some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested.
- Users must respect PubChem usage policies: no more than 5 requests per second and a 30-second per-request time limit.

## Evidence

- [intro] PUG-View serves information needed to render interactive web pages and allows programmatic access to chemical annotations and summary information in PubChem.: "which serves information needed to render interactive web pages but that also allows one to programmatically access the chemical annotations and summary information in PubChem"
- [other] LCSS data for a compound are retrieved via a PUG-View data request with the optional toc parameter set to LCSS+TOC.: [evidence span withheld — anchor status: not_found]
- [other] Validation criteria: the output file exists, is valid JSON, and contains the LCSS-organized health and safety annotation data.: [evidence span withheld — anchor status: not_found]
- [discussion] PUG-View does not accept multiple identifiers.: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Some specialized tabular data are not accessible through PUG-View.: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
