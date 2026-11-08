---
name: pug-view-request-url-construction
description: Use when programmatic access to PubChem annotation/summary data is needed
  — e.g., LCSS chemical-safety sections or historical record versions — rather than
  computed compound properties.
when_to_use_negative:
- Retrieving specific individual computed compound properties — PUG-REST is best suited
  for that, not PUG-View
- Fetching annotation data for multiple record identifiers in one call — a PUG-View
  request cannot take multiple input record identifiers
- Retrieving specialized tabular data (e.g., certain tabular annotations) that come
  from other backend services and are not accessible through PUG-View
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: REST-style web service endpoint whose request URL is constructed and queried
    via HTTP GET to retrieve PubChem annotation data as JSON
- name: PUG-REST
  role: Alternative PubChem interface preferred for individual computed compound properties;
    contrast point for choosing PUG-View for annotation data
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:e6dc877b5e60a2e21b723531527d4200acaf63a433d039661183b27ce448907e
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
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pug-view-request-url-construction

## Summary

Constructs PUG-View REST-style request URLs to retrieve PubChem annotation content (e.g., LCSS safety summaries, previous record versions) as JSON for a single compound, substance, or assay record.

## When to use

Use when programmatic access to PubChem annotation/summary data is needed — e.g., LCSS chemical-safety sections or historical record versions — rather than computed compound properties.

## When NOT to use

- Retrieving specific individual computed compound properties — PUG-REST is best suited for that, not PUG-View
- Fetching annotation data for multiple record identifiers in one call — a PUG-View request cannot take multiple input record identifiers
- Retrieving specialized tabular data (e.g., certain tabular annotations) that come from other backend services and are not accessible through PUG-View

## Inputs

- Single PubChem record identifier: CID (compound), SID (substance), or AID (assay)
- Optional parameter: toc (e.g., LCSS+TOC) to reorganize annotations by a specific table of contents
- Optional parameter: version (e.g., 2 for substance, 1.1 for assay) to retrieve a previous record version

## Outputs

- JSON document following the PUG-View annotation content model (same content model as the ASNT/ASNB formats), e.g., lcss_cid887.json, substance_46507068_version_2.json, assay_369_version_1.1.json

## How to apply

Formulate the request URL using the PUG-View pattern https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/{record_type}/{ID}/JSON, where record_type is compound, substance, or assay and only ONE record identifier may be encoded per request, since PUG-View cannot take multiple input record identifiers. Add optional parameters to tailor the response: toc=LCSS+TOC (space encoded as '+') reorganizes annotations under the Laboratory Chemical Safety Summary table of contents (e.g., compound CID 887), while version=n or version=n.m appended to substance (e.g., SID 46507068 with version=2) or assay (e.g., AID 369 with version=1.1) requests retrieves previous record versions, assay versions using the n.m form. Issue throttled HTTP GET requests (≤5 per second per PubChem usage policy, 30-second per-request time limit) and save the returned JSON, which follows the PUG-View annotation content model (same model as ASNT/ASNB formats). Validate that each saved file parses as valid JSON and contains the requested annotation sections or record version rather than an error message.

## Related tools

- **PUG-View** (REST-style web service endpoint whose request URL is constructed and queried via HTTP GET to retrieve PubChem annotation data as JSON)
- **PUG-REST** (Alternative PubChem interface preferred for individual computed compound properties; contrast point for choosing PUG-View for annotation data)

## Examples

```
curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/887/JSON?toc=LCSS+TOC" > lcss_cid887.json
```

## Evaluation signals

- Each saved response parses as valid JSON (e.g., json.load succeeds on lcss_cid887.json)
- Response contains the LCSS-organized health and safety annotation sections for the requested CID rather than the full Compound Summary content
- For version-parameterized requests, the returned record represents the requested version (e.g., version 2 of SID 46507068; version 1.1 of AID 369) rather than an error message
- Only a single record identifier is encoded in each request URL
- Request rate respects PubChem usage policy: no more than 5 requests per second and a 30-second per-request time limit

## Limitations

- A PUG-View request cannot take multiple input record identifiers
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless full annotation data retrieval is requested
- Some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View
- PubChem usage policies cap request rates at 5 per second with a 30-second per-request time limit

## Evidence

- [abstract] One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem.: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [discussion] A PUG-View request cannot take multiple input record identifiers.: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] PUG-View allows one to programmatically access this annotation content in PubChem.: "PUG-View allows one to programmatically access this annotation content in PubChem and is also used as a backend service"
- [intro] Existing interfaces were not designed for annotation data, motivating PUG-View.: "Because the existing web service interfaces in PubChem (including PUG-REST) were not designed to handle this type of data appropriately, PubChem developed a new REST-style interface"
