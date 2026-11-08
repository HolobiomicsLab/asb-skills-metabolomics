---
name: annotation-source-filtering
description: Use when a PUG-View bulk annotation request under a 'heading' path returns
  more entries than needed and you want to restrict results to one provider (e.g.
  source=DrugBank) rather than all annotation sources.
when_to_use_negative:
- You need annotation data under multiple headings for one compound in a single request
  — PUG-View cannot combine headings, and full annotation retrieval would be required
  instead.
- You need to pass multiple record identifiers in one request — a PUG-View request
  cannot take multiple input record identifiers.
- You need computed compound properties rather than curated annotation content — PUG-REST
  is the better fit for that data class.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: PUG-View
  role: REST-style web service used to issue the 'annotations/heading' request with
    the optional 'source' parameter and JSON output, and to read TotalPages/Pages
    metadata for pagination.
- name: PubChem
  role: Repository whose chemical annotation content is retrieved and whose usage
    limits (5 requests/second, 400/minute, 300 seconds running time) bound the filtering
    loop.
  repo: https://pubchem.ncbi.nlm.nih.gov
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:232e0447393cd55c92c0d1d8a185e5edd37f1e1f6d185e98c9cda0571a219cef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  - PubChem
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# annotation-source-filtering

## Summary

Restricts PUG-View annotation retrieval to a named data source by appending the optional 'source' parameter to an 'annotations/heading' request URL, so that only annotation entries contributed by a specific provider (e.g. DrugBank) are returned for a given heading across all PubChem records.

## When to use

Use when a PUG-View bulk annotation request under a 'heading' path returns more entries than needed and you want to restrict results to one provider (e.g. source=DrugBank) rather than all annotation sources.

## When NOT to use

- You need annotation data under multiple headings for one compound in a single request — PUG-View cannot combine headings, and full annotation retrieval would be required instead.
- You need to pass multiple record identifiers in one request — a PUG-View request cannot take multiple input record identifiers.
- You need computed compound properties rather than curated annotation content — PUG-REST is the better fit for that data class.

## Inputs

- PubChem heading name (e.g. 'Viscosity') for the annotation type of interest
- Desired data source name (e.g. 'DrugBank') to restrict retrieved annotation entries
- PUG-View request URL in the 'annotations/heading' form with JSON output

## Outputs

- Paginated JSON annotation entries under the given heading contributed solely by the specified source, including links to primary PubChem records
- TotalPages/Pages metadata used to exhaust the pagination loop

## How to apply

Construct a PUG-View request URL using the 'annotations/heading/<HEADING>/JSON' path without a record identifier, then append the 'source' optional parameter with the desired provider name (e.g. source=DrugBank) to filter the returned annotation entries to that data source only. Because heading names containing URL-incompatible special characters such as a forward slash must be passed as the 'heading' optional parameter instead of the URL path, the source filter can be combined with this alternative form without changing its semantics. Iterate over pages by appending the 'page' parameter up to the reported TotalPages count, respecting PubChem's usage limits (no more than 5 requests per second, 400 requests per minute, 300 seconds running time per minute). The rationale is that bulk annotation retrieval spans all records under a heading, so without a source filter the response mixes providers; the source parameter narrows the dataset before pagination is even started, which keeps each page response within the 30-second per-request time limit.

## Related tools

- **PUG-View** (REST-style web service used to issue the 'annotations/heading' request with the optional 'source' parameter and JSON output, and to read TotalPages/Pages metadata for pagination.)
- **PubChem** (Repository whose chemical annotation content is retrieved and whose usage limits (5 requests/second, 400/minute, 300 seconds running time) bound the filtering loop.) — https://pubchem.ncbi.nlm.nih.gov

## Examples

```
requests.get('https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/annotations/heading/Viscosity/JSON?source=DrugBank&page=1')
```

## Evaluation signals

- The response JSON reports TotalPages/Pages; the pagination loop should iterate from page 1 through TotalPages and no further.
- Every returned annotation entry must be attributable to the named source (e.g. all entries link to DrugBank-derived content), not a mix of providers.
- Requests stay within PubChem's documented limits: no more than 5 per second, 400 per minute, and 300 seconds of running time per minute.
- Heading names with URL-incompatible special characters (e.g. a forward slash) must appear as the 'heading' optional parameter rather than in the URL path; verify the URL is still valid and returns the same heading's annotations.

## Limitations

- A PUG-View request cannot take multiple input record identifiers, so this filter cannot be applied per-record batch.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless full annotation data retrieval is requested.
- Some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View, so the source filter cannot reach them.

## Evidence

- [full_text] Bulk retrieval of one annotation type: [evidence span withheld — anchor status: not_found]
- [full_text] optional source or heading parameters: [evidence span withheld — anchor status: not_found]
- [intro] pubchem developed pug-view: "PubChem developed a new REST-style interface, called PUG-View"
- [intro] pug-rest heavily used: "PUG-REST [4, 6, 7] is the most heavily used, with millions of daily requests from tens of thousands of unique IP addresses."
