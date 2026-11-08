---
name: json-parsing
description: Use when a PUG-View request (e.g. https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON)
  returns a JSON document that must be decoded into a Record object, saved to disk,
  or filtered to specific (sub)headings before downstream analysis.
when_to_use_negative:
- The retrieval target is computed compound properties rather than annotation data
  — PUG-REST is better suited for specific individual computed properties, and PUG-View
  responses are not designed for that data type.
- Multiple record identifiers need to be fetched in one HTTP call — a PUG-View request
  cannot take multiple input record identifiers, so batching must happen at the request
  level, not in a single parsed response.
- Tabular, specialized backend data not served through PUG-View is needed — such data
  are not currently accessible through PUG-View regardless of JSON parsing.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3345
- http://edamontology.org/topic_0091
tools:
- name: PUG-View
  role: REST-style web service interface whose JSON responses are the objects being
    parsed; provides full annotation data and the index (Table of Contents) requests
    for PubChem records.
  repo: https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/
- name: requests
  role: Python HTTP library used to issue the PUG-View GET request whose JSON body
    is then parsed.
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:3065d33b7975df49bd22df5756872d019fc8f01f7130dbc56f017a2fda24be87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  - requests
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3345
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# json-parsing

## Summary

Parses JSON responses returned by the PUG-View REST interface so that annotation content for PubChem compound, substance, or assay records can be programmatically consumed, validated, and reduced to the desired (sub)headings or full-record data.

## When to use

Use when a PUG-View request (e.g. https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON) returns a JSON document that must be decoded into a Record object, saved to disk, or filtered to specific (sub)headings before downstream analysis.

## When NOT to use

- The retrieval target is computed compound properties rather than annotation data — PUG-REST is better suited for specific individual computed properties, and PUG-View responses are not designed for that data type.
- Multiple record identifiers need to be fetched in one HTTP call — a PUG-View request cannot take multiple input record identifiers, so batching must happen at the request level, not in a single parsed response.
- Tabular, specialized backend data not served through PUG-View is needed — such data are not currently accessible through PUG-View regardless of JSON parsing.

## Inputs

- HTTP response body of a PUG-View request (raw JSON text)
- Record identifier(s) already encoded in the request URL (single compound CID, substance SID, or assay AID)
- PUG-View request URL string specifying 'data' or 'index' annotation type plus 'JSON' output format

## Outputs

- Parsed Python/JSON object exposing the top-level 'Record' node
- JSON files on disk: annotations_cid1983.json, annotations_sid46506142.json, annotations_aid1259416.json
- compound_toc.json containing the Table of Contents (list of available (sub)headings)
- Failure report for responses that return an error message (e.g. unknown record) instead of valid JSON

## How to apply

Issue the PUG-View HTTP GET request with the three required URL pieces—'data' annotation type, a single record identifier (compound/1983, substance/46506142, or assay/1259416), and 'JSON' as the output format—then decode the returned body with a JSON parser (e.g. Python's json.loads) so the top-level 'Record' node becomes accessible. Because PUG-View serves JSON by default and the response for a single record contains the entire record's annotation content, parse the tree and walk it with SectionName/Heading keys to extract only the desired (sub)headings instead of the full payload; for an index request (…/rest/pug_view/index/compound/1983/JSON) parse the returned Table of Contents that lists available (sub)headings without the full record data. Validate each parsed file by confirming it loads as valid JSON and contains the expected 'Record' node for the requested identifier; on parse errors (e.g. a 'unknown record' message) report the response as a failed retrieval. Persist the parsed result to a structured file (annotations_cid1983.json, annotations_sid46506142.json, annotations_aid1259416.json, or compound_toc.json) so the parsed JSON can be reused without re-issuing HTTP requests.

## Related tools

- **PUG-View** (REST-style web service interface whose JSON responses are the objects being parsed; provides full annotation data and the index (Table of Contents) requests for PubChem records.) — https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/
- **requests** (Python HTTP library used to issue the PUG-View GET request whose JSON body is then parsed.)

## Examples

```
import requests, json; r = requests.get('https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON'); rec = r.json()['Record']; json.dump(rec, open('annotations_cid1983.json','w'))
```

## Evaluation signals

- The parsed file loads with json.load without raising a decode error (valid JSON).
- The parsed object contains a 'Record' node whose identifier matches the requested CID/SID/AID (e.g. 1983, 46506142, 1259416).
- For an index request, the parsed output contains the Table of Contents — a list of (sub)headings — and does NOT contain the full record's annotation data content.
- Any response that fails to parse as JSON or contains an error message (e.g. 'unknown record') is flagged as a failed retrieval and reported, not silently accepted.
- The parsed output is saved to the expected structured filename (annotations_cid1983.json / annotations_sid46506142.json / annotations_aid1259416.json / compound_toc.json) and matches the annotation content served for the corresponding Summary/Record page.

## Limitations

- A PUG-View request cannot take multiple input record identifiers, so each identifier must be parsed from a separate HTTP response.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request unless the full annotation data retrieval is requested — partial heading retrieval requires separate parses of separate responses.
- Some specialized (especially tabular) data come from other backend services and are not currently accessible through PUG-View, so they cannot be parsed from these responses.
- Responses may return an error message (e.g. unknown record) instead of valid JSON, which must be detected during parsing and reported as a failure.

## Evidence

- [methods] Full annotation content for a single record is retrieved by a PUG-View URL that embeds the database, record identifier, and output format, e.g. https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON: [evidence span withheld — anchor status: not_found]
- [results] PUG-View provides an index request that returns the Table of Contents for a record's Summary page, listing the available (sub)headings without the full record data content.: [evidence span withheld — anchor status: not_found]
- [discussion] a PUG-View request cannot take multiple input record identifiers: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
