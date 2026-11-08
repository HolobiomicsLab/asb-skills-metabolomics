---
name: http-request-throttling
description: Use when a script iterates over many PUG-View URLs — pagination pages
  of a bulk heading request or one request per record identifier — and must avoid
  tripping PubChem's 5 req/s, 400 req/min, 300 s/min usage limits.
when_to_use_negative:
- The task is a single one-off PUG-View request (e.g. one CID's JSON full-record retrieval);
  with no loop there is nothing to throttle.
- The needed data are specialized (especially tabular) data served by other backend
  service(s) that are not currently accessible through PUG-View.
- You expect annotation data under multiple headings for one compound in a single
  call — instead request the full annotation data retrieval; throttling only paces
  the per-request loop, it does not merge heading queries.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3311
tools:
- name: PUG-View
  role: The REST-style web service whose 30-second per-request time limit and usage
    limits (5 req/s, 400/min, 300 s running time per minute) dictate the throttling
    and pagination strategy
  repo: https://pubchem.ncbi.nlm.nih.gov
- name: PubChem
  role: Data repository whose annotation content is being downloaded under the rate-limit
    policy
  repo: https://pubchem.ncbi.nlm.nih.gov
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:e50212a298d023624772b5f919d4caa22a9d6bb7218b63148706a734c4db8195
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  - PubChem
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3311
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# http-request-throttling

## Summary

Paces loops of HTTP requests against PubChem's PUG-View REST service so bulk annotation retrieval stays inside PubChem's published usage limits (no more than 5 requests per second, 400 per minute, 300 s running time per minute). Typically applied while paginating through large annotation sets such as a heading-wide download (e.g. /annotations/heading/Viscosity/JSON).

## When to use

Use when a script iterates over many PUG-View URLs — pagination pages of a bulk heading request or one request per record identifier — and must avoid tripping PubChem's 5 req/s, 400 req/min, 300 s/min usage limits.

## When NOT to use

- The task is a single one-off PUG-View request (e.g. one CID's JSON full-record retrieval); with no loop there is nothing to throttle.
- The needed data are specialized (especially tabular) data served by other backend service(s) that are not currently accessible through PUG-View.
- You expect annotation data under multiple headings for one compound in a single call — instead request the full annotation data retrieval; throttling only paces the per-request loop, it does not merge heading queries.

## Inputs

- PUG-View request URLs (e.g. https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON or /rest/pug_view/annotations/heading/Viscosity/JSON)
- First-page JSON response containing the 'TotalPage'/'TotalPages' and 'Pages' pagination fields
- Page indices / page parameter values to iterate over (page=2, page=3, ... up to TotalPages)

## Outputs

- Concatenated JSON annotation entries (with links to primary PubChem records) accumulated across all paginated requests
- Per-record saved JSON files such as annotations_cid1983.json, annotations_sid46506142.json, annotations_aid1259416.json
- Failure report for any request that returned an error message (e.g., unknown record)

## How to apply

Wrap every PUG-View call in a paced loop: send the first request (e.g. https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/annotations/heading/Viscosity/JSON) using the 'requests' library, read the returned 'TotalPage'/'TotalPages' count and current 'Pages' number, then loop over remaining pages by appending the 'page' optional parameter (page=2, page=3, ...) to the same URL. The rationale for pagination is that PUG-View imposes a 30-second per-request time limit, so a throttled multi-page loop circumvents it while still respecting the usage policy. Insert sleep/delay logic between requests so the client never exceeds 5 requests per second, 400 requests per minute, or 300 s of running time per minute (a simple time.sleep(0.3) between calls satisfies the 5 req/s bound). Optionally refine each throttled request with the 'source' parameter (e.g. source=DrugBank for LogP data) or pass special-character heading names via the 'heading' parameter instead of the URL path. Accumulate annotation entries plus their links to primary PubChem records from every page into a single dataset, and report any request that returns an error message as a failure.

## Related tools

- **PUG-View** (The REST-style web service whose 30-second per-request time limit and usage limits (5 req/s, 400/min, 300 s running time per minute) dictate the throttling and pagination strategy) — https://pubchem.ncbi.nlm.nih.gov
- **PubChem** (Data repository whose annotation content is being downloaded under the rate-limit policy) — https://pubchem.ncbi.nlm.nih.gov

## Examples

```
import requests, time; [requests.get(f"https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/annotations/heading/Viscosity/JSON?page={p}") for p in range(1, TP+1)]; time.sleep(0.3)
```

## Evaluation signals

- Request timestamps show no more than 5 requests in any 1-second window and no more than 400 requests in any rolling minute
- Cumulative request running time stays within 300 s per minute of wall-clock time
- The pagination loop ran until page coverage spanned the reported TotalPages count, confirming all pages were exhausted
- Every saved JSON file parses as valid JSON, contains the 'Record' node for the requested identifier, and matches the annotation content of the corresponding PubChem Summary/Record page
- Requests returning error messages (e.g., unknown record) are reported as failures rather than silently skipped

## Limitations

- PUG-View enforces a 30-second per-request time limit, so very large annotation sets must be split into paginated requests rather than fetched at once
- A PUG-View request cannot take multiple input record identifiers, so bulk collection requires one throttled request per record
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request unless the full annotation data retrieval is requested
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View at all

## Evidence

- [methods] Throttle the loop to respect PubChem usage limits (no more than 5 requests per second, 400 per minute, 300 s running time per minute).: [evidence span withheld — anchor status: not_found]
- [methods] page-optional-parameter pagination rationale: [evidence span withheld — anchor status: not_found]
- [discussion] one-identifier-per-request constraint: "a PUG-View request cannot take multiple input record identifiers"
- [intro] shared-service load rationale: "PUG-REST [4, 6, 7] is the most heavily used, with millions of daily requests from tens of thousands of unique IP addresses."
