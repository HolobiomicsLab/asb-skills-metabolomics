---
name: pubchem-heading-url-encoding
description: Use when you need one annotation type (identified by heading name) for
  all PubChem records via PUG-View, especially when the heading contains URL-incompatible
  characters (e.g. '/'), or when results must be paginated or filtered by source.
when_to_use_negative:
- When you need to supply multiple input record identifiers in a single request —
  a PUG-View request cannot take multiple input record identifiers
- When you need annotation data under multiple headings for one compound in a single
  request — this is not supported unless full annotation data retrieval is requested
- When the data of interest is specialized tabular content that comes from other backend
  services and is not accessible through PUG-View
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: REST-style web service interface specialized for accessing PubChem annotation
    data; its 'annotations/heading' URL syntax and optional 'heading', 'page' and
    'source' parameters are the target of URL construction
- name: requests
  role: Python HTTP library used to send the paginated heading-specified PUG-View
    requests and parse the returned JSON
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:fd9ec69f63e1f4d2f6afc9540fd2c79c2c3c684650bc62568eb1154d2a74c7c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  - PubChem
  - requests
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pubchem-heading-url-encoding

## Summary

Constructs PUG-View REST request URLs that encode an annotation heading directly in the URL path (e.g. /annotations/heading/Viscosity/JSON) or via the 'heading' optional parameter, enabling bulk retrieval of one annotation type for all PubChem records with pagination and source filtering.

## When to use

Use when you need one annotation type (identified by heading name) for all PubChem records via PUG-View, especially when the heading contains URL-incompatible characters (e.g. '/'), or when results must be paginated or filtered by source.

## When NOT to use

- When you need to supply multiple input record identifiers in a single request — a PUG-View request cannot take multiple input record identifiers
- When you need annotation data under multiple headings for one compound in a single request — this is not supported unless full annotation data retrieval is requested
- When the data of interest is specialized tabular content that comes from other backend services and is not accessible through PUG-View

## Inputs

- Annotation heading name (e.g. 'Viscosity', 'LogP'), supplied as URL path component or 'heading' optional parameter when it contains special characters like '/'
- Optional 'page' parameter for pagination (e.g. page=2 up to TotalPages)
- Optional 'source' parameter to filter by data source (e.g. DrugBank)

## Outputs

- JSON response containing annotation entries of the requested heading for all PubChem records, each with links to primary PubChem records
- Pagination metadata fields TotalPages (or TotalPage) and Pages read from the JSON response
- Concatenated dataset spanning all returned pages

## How to apply

Build a PUG-View URL with the 'annotations/heading' path, placing the heading name (e.g. Viscosity) as the ANNOTATION component and JSON as the output format, so no record identifier appears in the request. If the heading contains URL-incompatible special characters such as a forward slash, pass it instead as the optional 'heading' parameter. Send the request (e.g. with the Python 'requests' library), parse the JSON for 'TotalPages'/'TotalPage' and the current 'Pages' number, then loop over remaining pages by appending the 'page' optional parameter (page=2, page=3, ...) — this also works around PUG-View's 30-second per-request time limit on large annotation sets. Throttle the loop to PubChem's usage limits (no more than 5 requests per second, 400 per minute, 300 s running time per minute) and optionally add a 'source' parameter (e.g. source=DrugBank) to restrict annotation data to a specific data source. Accumulate annotation entries and their links to primary PubChem records from every page into one dataset.

## Related tools

- **PUG-View** (REST-style web service interface specialized for accessing PubChem annotation data; its 'annotations/heading' URL syntax and optional 'heading', 'page' and 'source' parameters are the target of URL construction)
- **requests** (Python HTTP library used to send the paginated heading-specified PUG-View requests and parse the returned JSON)

## Examples

```
import requests; r=requests.get('https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/annotations/heading/Viscosity/JSON', params={'page':2})
```

## Evaluation signals

- The JSON response parses successfully and reports TotalPages and Pages values consistent with the requested heading
- The pagination loop's collected pages span all pages from 1 to TotalPages, confirming the loop exhausted all returned pages
- Every accumulated annotation entry carries a link to its primary PubChem record
- Headings containing URL-incompatible characters (e.g. '/') are successfully retrieved when passed via the 'heading' optional parameter rather than the URL path
- Request rate stays within PubChem limits (≤5 requests/second, ≤400/minute, ≤300 s running time per minute)

## Limitations

- PUG-View enforces a 30-second per-request time limit, so large annotation sets must be retrieved page-by-page via the 'page' optional parameter
- Heading names containing URL-incompatible special characters such as a forward slash cannot be placed in the URL path and must use the 'heading' optional parameter
- A PUG-View request cannot take multiple input record identifiers, and annotation data under multiple headings for a given compound cannot be retrieved with a single request unless full annotation retrieval is requested
- Some specialized (especially tabular) data are not currently accessible through PUG-View
- PubChem usage limits apply: no more than 5 requests per second, 400 per minute, and 300 s running time per minute

## Evidence

- [abstract] PUG-View REST annotation retrieval: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [intro] Why PUG-View exists: "Because the existing web service interfaces in PubChem (including PUG-REST) were not designed to handle this type of data appropriately, PubChem developed a new REST-style interface, called PUG-View"
- [discussion] No multiple record identifiers per request: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] One heading per request: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
