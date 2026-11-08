---
name: rest-api-pagination-looping
description: Use when a PUG-View bulk request must fetch one annotation heading (e.g.
  Viscosity, LogP) for all records with no record identifier in the URL, and the response
  reports TotalPages > 1 requiring paged retrieval.
when_to_use_negative:
- The request needs annotation data under multiple headings for a single compound
  — PUG-View cannot combine headings in one request unless full annotation retrieval
  is requested
- The request must pass multiple input record identifiers — a PUG-View request cannot
  take multiple record identifiers
- The target is specialized (especially tabular) data served by other PubChem backend
  services that is not currently accessible through PUG-View
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: REST-style web service that serves the /rest/pug_view/annotations/heading/...
    bulk endpoints; provides TotalPages/Pages fields and the page and heading optional
    parameters used to paginate
- name: requests
  role: Python HTTP library used to send each paged PUG-View request and parse the
    returned JSON
- name: PubChem
  role: Repository whose records and annotation content (e.g. viscosity, LogP from
    DrugBank) are being retrieved via PUG-View
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:419b20428bd537d5c1a342921f58d31fe3cb646eb40c7be6095a7d28433d4dfd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  - PubChem
  - requests
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# rest-api-pagination-looping

## Summary

Retrieves one annotation type for ALL PubChem records via PUG-View by looping a heading-specified URL (e.g. /rest/pug_view/annotations/heading/Viscosity/JSON) over pages, using the TotalPages/Pages fields and the page optional parameter to work around the 30-second per-request time limit.

## When to use

Use when a PUG-View bulk request must fetch one annotation heading (e.g. Viscosity, LogP) for all records with no record identifier in the URL, and the response reports TotalPages > 1 requiring paged retrieval.

## When NOT to use

- The request needs annotation data under multiple headings for a single compound — PUG-View cannot combine headings in one request unless full annotation retrieval is requested
- The request must pass multiple input record identifiers — a PUG-View request cannot take multiple record identifiers
- The target is specialized (especially tabular) data served by other PubChem backend services that is not currently accessible through PUG-View

## Inputs

- PUG-View bulk request URL of the form https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/annotations/heading/<HeadingName>/JSON (no record identifier)
- Heading name string; use the 'heading' optional parameter when the name contains URL-incompatible characters such as '/'
- Optional 'page' parameter values (2..TotalPages) for the loop
- Optional 'source' parameter (e.g. DrugBank) to filter annotations by data source

## Outputs

- Per-page JSON responses containing the Pages number and TotalPages count
- A single concatenated dataset of annotation entries of the requested heading for all records
- Links from each annotation entry to the primary PubChem records it annotates

## How to apply

Construct a PUG-View URL using the 'annotations/heading' path with the heading name (e.g. Viscosity) and JSON output, omitting any record identifier; if the heading contains URL-incompatible characters such as a forward slash, pass it instead as the 'heading' optional parameter. Send the request with the 'requests' HTTP library and parse the JSON to read the TotalPages count and the current Pages number. Loop over subsequent pages by appending page=2, page=3, ... up to TotalPages to the same URL — paginating this way circumvents the 30-second per-request time limit on large annotation sets. Throttle the loop to respect PubChem usage limits (no more than 5 requests per second, 400 per minute, and 300 s running time per minute). Optionally add the 'source' parameter to restrict the data to one source (e.g. source=DrugBank for LogP data). Accumulate annotation entries, including their links to primary PubChem records, from every page and concatenate them into a single dataset.

## Related tools

- **PUG-View** (REST-style web service that serves the /rest/pug_view/annotations/heading/... bulk endpoints; provides TotalPages/Pages fields and the page and heading optional parameters used to paginate)
- **requests** (Python HTTP library used to send each paged PUG-View request and parse the returned JSON)
- **PubChem** (Repository whose records and annotation content (e.g. viscosity, LogP from DrugBank) are being retrieved via PUG-View)

## Examples

```
import requests; r = requests.get("https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/annotations/heading/Viscosity/JSON"); total = r.json()["TotalPages"]
```

## Evaluation signals

- Page coverage of the collected JSON spans the reported TotalPages count, confirming the pagination loop exhausted all returned pages
- The Pages value returned increments 1, 2, 3, ... with each request and the final page equals TotalPages
- The loop respects PubChem usage limits: no more than 5 requests per second, 400 per minute, and 300 s running time per minute
- Every annotation entry in the accumulated dataset carries links to its primary PubChem record(s)
- Headings containing '/' appear in the dataset, proving they were passed via the heading= parameter rather than the URL path

## Limitations

- PUG-View enforces a 30-second per-request time limit, which is why large annotation sets must be paged rather than fetched in one call
- PubChem usage policies cap traffic at 5 requests per second, 400 requests per minute, and 300 s running time per minute
- A PUG-View request cannot take multiple input record identifiers, and annotation data under multiple headings for a given compound cannot be retrieved in a single request unless full annotation retrieval is requested
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View

## Evidence

- [abstract] PUG-View is a REST interface specialized for annotation retrieval: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [intro] PUG-View was created because PUG-REST could not handle annotation data: "Because the existing web service interfaces in PubChem (including PUG-REST) were not designed to handle this type of data appropriately, PubChem developed a new REST-style interface, called PUG-View"
- [discussion] PUG-View has no multiple-record-identifier input: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Only one heading per request unless full retrieval is used: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
- [discussion] Some tabular data are not accessible through PUG-View: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
