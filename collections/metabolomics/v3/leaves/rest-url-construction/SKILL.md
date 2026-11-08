---
name: rest-url-construction
description: Use when you need to retrieve a specific annotation for a given PubChem
  compound record through PUG-View but must first know how to form the request URL.
when_to_use_negative:
- When you need to retrieve annotation data for multiple compounds in a single request
  — PUG-View cannot take multiple input record identifiers.
- When you need annotation data under multiple headings for a given compound without
  requesting the full annotation data — a single request cannot do this.
- When you need specific individual computed compound properties — PUG-REST is best
  for that.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0154
tools:
- name: PUG-View
  role: The REST-style web service interface whose request URL is constructed by this
    skill; provides annotation data, Table of Contents (index), and summary information
    for PubChem records.
- name: requests
  role: Python HTTP library used to issue GET requests against the constructed PUG-View
    URL.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:eeb96ed712b84fd51cd6f9fc6f5ee00c7eae0b25b9b121ed361ce0554b74eefb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  - requests
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rest-url-construction

## Summary

Constructs a PUG-View request URL by encoding three pieces of information: the annotation type, the record specification (database type and record identifier), and the desired output format.

## When to use

Use when you need to retrieve a specific annotation for a given PubChem compound record through PUG-View but must first know how to form the request URL.

## When NOT to use

- When you need to retrieve annotation data for multiple compounds in a single request — PUG-View cannot take multiple input record identifiers.
- When you need annotation data under multiple headings for a given compound without requesting the full annotation data — a single request cannot do this.
- When you need specific individual computed compound properties — PUG-REST is best for that.

## Inputs

- PubChem database type and a single record identifier (e.g. compound/1983)
- PUG-View annotation type (e.g. 'index', 'summary', or a specific heading such as 'Safety and Hazards')
- Desired output format (e.g. 'JSON')

## Outputs

- PUG-View request URL string of the form https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/{annotation_type}/{record_specification}/{output_format}
- JSON response from the PUG-View request, parsed into structured data (e.g. compound_toc.json for an index request)

## How to apply

Build a REST-style URL of the form https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/{annotation_type}/{record_specification}/{output_format}, where annotation_type identifies the annotation heading (e.g. 'index', 'summary', or a specific heading such as 'Safety and Hazards'), record_specification gives the database type and a single record identifier (e.g. 'compound/1983'), and output_format is 'JSON'. The URL must encode exactly one input record identifier because a PUG-View request cannot take multiple input record identifiers. Use the index annotation type when you need the Table of Contents of available (sub)headings without the full annotation data, as in .../rest/pug_view/index/compound/1983/JSON.

## Related tools

- **PUG-View** (The REST-style web service interface whose request URL is constructed by this skill; provides annotation data, Table of Contents (index), and summary information for PubChem records.)
- **requests** (Python HTTP library used to issue GET requests against the constructed PUG-View URL.)

## Examples

```
requests.get('https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/index/compound/1983/JSON')
```

## Evaluation signals

- Verify the URL contains exactly three path components after /rest/pug_view/ — annotation type, record specification, and output format.
- Confirm the record specification contains exactly one record identifier; a multi-identifier URL should fail because PUG-View cannot take multiple input record identifiers.
- For an index request, confirm the JSON response lists available (sub)headings (the Table of Contents) without the full record data content.
- Confirm the HTTP response is valid JSON before parsing; save parsed output to a structured file such as compound_toc.json.

## Limitations

- A PUG-View request cannot take multiple input record identifiers — batch retrieval requires one request per record.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request unless the full annotation data retrieval is requested.
- Some specialized (especially tabular) data are not currently accessible through PUG-View.

## Evidence

- [methods] three pieces of information: [evidence span withheld — anchor status: not_found]
- [methods] single record identifier: "a PUG-View request cannot take multiple input record identifiers"
- [results] index request returns Table of Contents: [evidence span withheld — anchor status: not_found]
- [methods] REST-style URL form: "https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/index/compound/1983/JSON"
- [abstract] PUG-View specialized for annotation data: "PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem"
