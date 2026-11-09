---
name: pug-view-bulk-annotation-retrieval
description: Use when you need one annotation type (e.g. Viscosity, LogP) for all
  PubChem records and the heading name is supplied with no record identifier in the
  request URL.
when_to_use_negative:
- The request needs multiple input record identifiers — a PUG-View request cannot
  take multiple input record identifiers, so per-record retrieval or PUG-REST should
  be used instead.
- Annotation data under multiple headings is needed for a given compound in one call
  — this is only possible if the full annotation data retrieval is requested, not
  via a heading-specified bulk URL.
- Tabular specialized data that come from other backend services and are not currently
  accessible through PUG-View.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0154
tools:
- name: PUG-View
  role: REST-style web service used to issue heading-specified bulk annotation requests
    and handle pagination via TotalPages/Pages with optional page, source, and heading
    parameters
  repo: https://pubchem.ncbi.nlm.nih.gov
- name: PubChem
  role: Underlying chemical data repository whose annotation records are retrieved
    through the PUG-View bulk endpoint
  repo: https://pubchem.ncbi.nlm.nih.gov
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:a34d8f7cbe209d00038c8f04fb8c2fe8670f5e2c77e6183d8b13de20d0249ded
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  - PubChem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pug-view-bulk-annotation-retrieval

## Summary

Retrieves a single annotation type (heading) for all PubChem records via PUG-View's heading-specified URL, paginating with the TotalPages/Pages count and optional page/source/heading parameters to bypass the 30-second per-request time limit.

## When to use

Use when you need one annotation type (e.g. Viscosity, LogP) for all PubChem records and the heading name is supplied with no record identifier in the request URL.

## When NOT to use

- The request needs multiple input record identifiers — a PUG-View request cannot take multiple input record identifiers, so per-record retrieval or PUG-REST should be used instead.
- Annotation data under multiple headings is needed for a given compound in one call — this is only possible if the full annotation data retrieval is requested, not via a heading-specified bulk URL.
- Tabular specialized data that come from other backend services and are not currently accessible through PUG-View.

## Inputs

- Desired annotation heading name (e.g. Viscosity)
- Optional output format specifier (JSON) for the PUG-View request URL
- Optional 'source' parameter to filter by data source (e.g. DrugBank)

## Outputs

- Paginated JSON containing annotation entries for the heading across all PubChem records
- Concatenated dataset of annotation entries plus links to primary PubChem records

## How to apply

Build a PUG-View bulk request URL using the 'annotations/heading' path with the heading name (e.g. /annotations/heading/Viscosity/JSON) as the ANNOTATION component and JSON as the output; if the heading contains URL-incompatible special characters such as a forward slash, pass it instead as the optional 'heading' parameter. Parse the returned JSON for TotalPages/Pages and loop over subsequent pages by appending the 'page' parameter (page=2, page=3, ...) to the same URL, which circumvents PUG-View's 30-second per-request time limit. Throttle the loop to PubChem's usage limits of no more than 5 requests per second, 400 requests per minute, and 300 seconds running time per minute. Optionally refine retrieval by adding the 'source' parameter (e.g. source=DrugBank) to restrict annotation data to a specific data source. Accumulate annotation entries and their links to primary PubChem records from every page into a single dataset.

## Related tools

- **PUG-View** (REST-style web service used to issue heading-specified bulk annotation requests and handle pagination via TotalPages/Pages with optional page, source, and heading parameters) — https://pubchem.ncbi.nlm.nih.gov
- **PubChem** (Underlying chemical data repository whose annotation records are retrieved through the PUG-View bulk endpoint) — https://pubchem.ncbi.nlm.nih.gov

## Examples

```
for page in range(1, TotalPages+1): r = requests.get(f'https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/annotations/heading/Viscosity/JSON?page={page}')
```

## Evaluation signals

- The collected JSON contains annotation entries whose page coverage spans the reported TotalPages count, confirming the pagination loop exhausted all returned pages.
- Each per-page response stays within PUG-View's 30-second per-request time limit and the throttled loop respects 5 requests/second, 400 requests/minute, and 300 seconds running time per minute.
- Annotation entries in the output include links to their primary PubChem records.
- Records with headings containing URL-incompatible special characters (e.g. a forward slash) are successfully retrieved via the optional 'heading' parameter rather than the path-embedded heading name.

## Limitations

- A PUG-View request cannot take multiple input record identifiers.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request unless full annotation data retrieval is requested.
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View.
- Per-request time limit is 30 seconds, so large annotation sets must be paginated.

## Evidence

- [full_text] Heading-specified URL semantics: [evidence span withheld — anchor status: not_found]
- [full_text] Pagination mechanism: [evidence span withheld — anchor status: not_found]
- [abstract] Purpose of the interface: "PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem"
- [discussion] Multi-record restriction: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Tabular data exclusion: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
