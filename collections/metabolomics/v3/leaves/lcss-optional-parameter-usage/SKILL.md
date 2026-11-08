---
name: lcss-optional-parameter-usage
description: Use when you need the health/safety (LCSS) annotation content for a single
  PubChem compound record, organized by the LCSS table of contents, as a JSON payload
  from PUG-View.
when_to_use_negative:
- You need to query multiple compound record identifiers in one call — a PUG-View
  request cannot take multiple input record identifiers
- You need specific individual computed compound properties — PUG-REST is best suited
  for that, not PUG-View
- You need annotation data under multiple headings for a compound in a single request
  (without retrieving the full annotation data), or specialized tabular data that
  come from other backend services and are not currently accessible through PUG-View
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: REST-style web service interface used to issue the compound/JSON data request
    with the optional toc=LCSS+TOC parameter and return the LCSS-organized annotation
    record
  repo: https://pubchem.ncbi.nlm.nih.gov/rest/pug_view
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:089e455e82758cee520e5f97aad639ce2512b66f532ca81944531e220dd626d3
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
yes_rate: 0.9
n_claims: 10
---

# lcss-optional-parameter-usage

## Summary

Retrieves a compound's Laboratory Chemical Safety Summary (LCSS) annotations from PubChem via a PUG-View data request using the optional toc parameter set to LCSS+TOC, which reorganizes the record annotations by the LCSS table of contents instead of the default one.

## When to use

Use when you need the health/safety (LCSS) annotation content for a single PubChem compound record, organized by the LCSS table of contents, as a JSON payload from PUG-View.

## When NOT to use

- You need to query multiple compound record identifiers in one call — a PUG-View request cannot take multiple input record identifiers
- You need specific individual computed compound properties — PUG-REST is best suited for that, not PUG-View
- You need annotation data under multiple headings for a compound in a single request (without retrieving the full annotation data), or specialized tabular data that come from other backend services and are not currently accessible through PUG-View

## Inputs

- A single PubChem Compound ID (CID), e.g. 887 (methanol)
- PUG-View request URL template: https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/{CID}/JSON?toc=LCSS+TOC

## Outputs

- JSON document (e.g. lcss_cid887.json) following the PUG-View annotation content model, containing LCSS-organized health and safety annotation sections for the compound

## How to apply

Formulate a PUG-View data request URL with the pattern https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/{CID}/JSON and append the optional parameter toc=LCSS+TOC, encoding the space as the plus character; this tells PUG-View to reorganize the record annotations according to the Laboratory Chemical Safety Summary table of contents rather than the default table of contents. Encode only a single record identifier (e.g. CID 887 for methanol), because a PUG-View request cannot take multiple input record identifiers. Issue the HTTP GET request, staying within PubChem usage policies (no more than 5 requests per second, 30-second per-request time limit). Parse the returned JSON, which follows the PUG-View annotation content model (the same model as the ASNT/ASNB formats), and confirm it contains the LCSS-organized health and safety sections rather than the full Compound Summary content. Write the payload to an output file (e.g. lcss_cid887.json) for downstream use.

## Related tools

- **PUG-View** (REST-style web service interface used to issue the compound/JSON data request with the optional toc=LCSS+TOC parameter and return the LCSS-organized annotation record) — https://pubchem.ncbi.nlm.nih.gov/rest/pug_view

## Examples

```
curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/887/JSON?toc=LCSS+TOC" > lcss_cid887.json
```

## Evaluation signals

- The output file (e.g. lcss_cid887.json) exists and parses as valid JSON
- The JSON follows the PUG-View annotation content model (same model as the ASNT/ASNB formats) rather than a PUG-REST property response
- The returned document contains the LCSS-organized health and safety annotation sections for the queried CID (e.g. methanol, CID 887), not the full Compound Summary content
- Only one record identifier was encoded in the request URL (PUG-View rejects multiple identifiers)
- Request rate stayed within PubChem usage policies: no more than 5 requests per second and a 30-second per-request time limit

## Limitations

- A PUG-View request cannot take multiple input record identifiers — one request per record
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View
- Requests are subject to PubChem usage policies: no more than 5 requests per second and a 30-second per-request time limit

## Evidence

- [abstract] PUG-View REST-style interface for annotation data: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [discussion] single-identifier limitation: "a PUG-View request cannot take multiple input record identifiers"
- [other] LCSS toc parameter example: [evidence span withheld — anchor status: not_found]
- [other] usage policy limits: [evidence span withheld — anchor status: not_found]
