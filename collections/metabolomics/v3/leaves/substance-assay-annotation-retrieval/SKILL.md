---
name: substance-assay-annotation-retrieval
description: Use when a workflow needs the full set of third-party chemical annotations
  or assay descriptions for a specific PubChem SID or AID, and the required data are
  not exposed by other PubChem programmatic access routes.
when_to_use_negative:
- Input is a list of many record identifiers needing bulk retrieval — PUG-View cannot
  take multiple input record identifiers in one request.
- Goal is retrieving a small set of specific computed compound properties rather than
  full annotation content — PUG-REST is better suited for that purpose.
- Only a specific tabular data type is needed — some specialized (especially tabular)
  data come from other backend services and are not currently accessible through PUG-View.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_2258
tools:
- name: PUG-View
  role: REST-style web service interface used to programmatically access full annotation
    data for PubChem substance and assay records, including prior versions via the
    version parameter
- name: PUG-REST
  role: Complementary PubChem web service for accessing specific individual computed
    compound properties; explicitly contrasted with PUG-View when retrieving annotation
    content
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:d8f4bd5061ddf2232566566439888ef4be5d727f59a873b21cec77552e1de132
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# substance-assay-annotation-retrieval

## Summary

Retrieve complete annotation data for a given substance or assay record from PubChem by issuing a PUG-View request URL against the /rest/pug_view/data/.../JSON endpoint, honoring version parameters and usage policies.

## When to use

Use when a workflow needs the full set of third-party chemical annotations or assay descriptions for a specific PubChem SID or AID, and the required data are not exposed by other PubChem programmatic access routes.

## When NOT to use

- Input is a list of many record identifiers needing bulk retrieval — PUG-View cannot take multiple input record identifiers in one request.
- Goal is retrieving a small set of specific computed compound properties rather than full annotation content — PUG-REST is better suited for that purpose.
- Only a specific tabular data type is needed — some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View.

## Inputs

- PubChem substance identifier (SID) for a depositor-provided substance record
- PubChem assay identifier (AID) for an assay record
- Optional version number (integer for substances, n.m form for assays)

## Outputs

- JSON response containing the full annotation data for the specified substance or assay record
- JSON response containing the full annotation data for a prior version of the record, when a version parameter is supplied

## How to apply

Construct a PUG-View REST request URL with the pattern /rest/pug_view/data/substance/{SID}/JSON or /rest/pug_view/data/assay/{AID}/JSON to fetch full annotation content for the given record identifier; unlike PUG-REST, which is optimized for specific individual computed compound properties, PUG-View is the correct interface for this kind of annotation and summary data because existing PubChem web services were not designed to handle it. To retrieve a historical version, append the optional version parameter to the URL: ?version=2 for a substance SID (e.g., 46507068) or ?version=1.1 for an assay AID (e.g., 369), since assay versions use the n.m form. Throttle requests to at most 5 requests per second per PubChem usage policy, and save the returned JSON to disk for downstream parsing. Note that a single PUG-View request cannot take multiple record identifiers, so issue one request per SID/AID; if only a single heading is needed, consider narrowing the request rather than pulling the full annotation blob.

## Related tools

- **PUG-View** (REST-style web service interface used to programmatically access full annotation data for PubChem substance and assay records, including prior versions via the version parameter)
- **PUG-REST** (Complementary PubChem web service for accessing specific individual computed compound properties; explicitly contrasted with PUG-View when retrieving annotation content)

## Examples

```
curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/substance/46507068/JSON?version=2" -o substance_46507068_version_2.json
```

## Evaluation signals

- HTTP GET returns a valid JSON object, not an error message, for the given SID or AID.
- Response matches the requested record: the substance response corresponds to the input SID, and the assay response corresponds to the input AID.
- When a version parameter is supplied (e.g., ?version=2 for a substance, ?version=1.1 for an assay), the returned data reflect that specific historical version.
- Request rate stays at or below the PubChem usage policy limit of 5 requests per second.
- Each request targets exactly one record identifier, since PUG-View cannot take multiple input record identifiers in a single request.

## Limitations

- A PUG-View request cannot take multiple input record identifiers.
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested.
- PUG-View is subject to PubChem usage policies, including a limit of no more than 5 requests per second.

## Evidence

- [other] PUG-View retrieves previous versions of substance or assay records by passing the version number as an optional parameter in the request URL: [evidence span withheld — anchor status: not_found]
- [full_text] full annotation data retrieval is requested: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
- [abstract] PUG-View is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [intro] PUG-View was created because existing PubChem web services were not designed for annotation data: "Because the existing web service interfaces in PubChem (including PUG-REST) were not designed to handle this type of data appropriately, PubChem developed a new REST-style interface, called PUG-View"
