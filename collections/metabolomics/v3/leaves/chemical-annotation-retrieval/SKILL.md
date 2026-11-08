---
name: chemical-annotation-retrieval
description: Use when you need programmatic access to PubChem's chemical annotations
  or summary information (e.g., LCSS, solubility, experimental properties) for a single
  record, and PUG-REST is unsuitable for such annotation data.
when_to_use_negative:
- You need annotations under multiple headings for one compound in a single request
  — this is not possible unless full annotation data retrieval is requested
- You need specific individual computed compound properties (e.g., molecular weight,
  XLogP) — PUG-REST is best suited for that
- You need specialized tabular data — some of it comes from other backend services
  and is not currently accessible through PUG-View
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_2258
tools:
- name: PUG-View
  role: REST-style web service interface specialized for accessing annotation data
    contained in PubChem; used to issue data requests scoped by heading or toc parameters
- name: PUG-REST
  role: Alternative PubChem interface referenced for contrast — best for accessing
    specific individual computed compound properties, not annotation content
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:52bfabfcc9f98513899615686f05b494284ae8fabd903f0772b3ea199cf6b88e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chemical-annotation-retrieval

## Summary

Retrieves PubChem annotation content (textual and summary data such as solubility, experimental properties, or LCSS safety sections) for a single record via the PUG-View REST-style interface, using heading or toc parameters to scope the request.

## When to use

Use when you need programmatic access to PubChem's chemical annotations or summary information (e.g., LCSS, solubility, experimental properties) for a single record, and PUG-REST is unsuitable for such annotation data.

## When NOT to use

- You need annotations under multiple headings for one compound in a single request — this is not possible unless full annotation data retrieval is requested
- You need specific individual computed compound properties (e.g., molecular weight, XLogP) — PUG-REST is best suited for that
- You need specialized tabular data — some of it comes from other backend services and is not currently accessible through PUG-View

## Inputs

- Single PubChem record identifier (e.g., CID 1983 or CID 887); PUG-View does not accept multiple identifiers
- Desired (sub)heading name (e.g., 'Solubility', 'Experimental Properties') or table of contents scope such as LCSS+TOC
- Output format choice (JSON or XML)

## Outputs

- Valid JSON/XML document following the PUG-View annotation content model, restricted to the requested heading or LCSS table of contents (e.g., solubility_cid1983.json, lcss_cid887.json)

## How to apply

Formulate a PUG-View request URL encoding the three required pieces of information: annotation type ('data'), the single record identifier (PUG-View does not accept multiple identifiers, e.g., /compound/1983/JSON), and the output format (JSON or XML). To restrict output to one (sub)heading, append the optional heading parameter (e.g., ?heading=Experimental+Properties or ?heading=Solubility) with spaces URL-encoded as '+'; to reorganize annotations under the Laboratory Chemical Safety Summary table of contents, use the toc parameter set to LCSS+TOC (e.g., ?toc=LCSS+TOC for CID 887). Optionally verify a heading exists first via an 'index' request that returns the record's Table of Contents without full data. Issue the HTTP GET to https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/..., parse the PUG-View annotation content model (same model as the ASNT/ASNB formats), and save the response as JSON.

## Related tools

- **PUG-View** (REST-style web service interface specialized for accessing annotation data contained in PubChem; used to issue data requests scoped by heading or toc parameters)
- **PUG-REST** (Alternative PubChem interface referenced for contrast — best for accessing specific individual computed compound properties, not annotation content)

## Examples

```
curl 'https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON?heading=Solubility' -o solubility_cid1983.json
```

## Evaluation signals

- Output file is valid JSON (or XML) parseable against the PUG-View annotation content model
- Returned annotation content is restricted to the single requested (sub)heading (e.g., only Solubility data for CID 1983)
- For the toc=LCSS+TOC request, the response contains the LCSS-organized health and safety annotation sections rather than the full Compound Summary content
- URL contains exactly one record identifier, since PUG-View does not accept multiple input identifiers
- A request for a heading under which the compound has no data returns an error message rather than an empty annotation payload — confirm this is the expected behavior for absent sections

## Limitations

- A PUG-View request cannot take multiple input record identifiers — one request per record
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless full annotation data retrieval is requested
- Some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View
- Usage policies: no more than 5 requests per second and a 30-second per-request time limit
- If the compound has no data under the requested heading, the request returns an error message

## Evidence

- [abstract] PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem.: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [results] A single-record PUG-View request accepts the optional heading parameter appended to the record URL with spaces URL-encoded as '+'.: [evidence span withheld — anchor status: not_found]
- [results] LCSS data for a compound are retrieved via a PUG-View data request with the optional toc parameter set to LCSS+TOC.: [evidence span withheld — anchor status: not_found]
- [discussion] A PUG-View request cannot take multiple input record identifiers.: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] PUG-REST is best for accessing specific individual computed compound properties.: "while PUG-REST is best for accessing specific individual computed compound properties"
