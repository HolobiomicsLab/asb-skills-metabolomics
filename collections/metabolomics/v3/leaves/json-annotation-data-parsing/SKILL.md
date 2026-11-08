---
name: json-annotation-data-parsing
description: Use when you need the annotation content (e.g. experimental properties,
  solubility, etc.) under a single specified (sub)heading for one PubChem record,
  returned as JSON or XML.
when_to_use_negative:
- A PUG-View request cannot take multiple input record identifiers — do not use this
  skill when you need to retrieve annotations for many records at once.
- Annotation data under multiple headings for a given compound cannot be retrieved
  with a single request, unless the full annotation data retrieval is requested —
  do not use this skill when you need several distinct headings for one record.
- Do not use this skill for specialized (especially tabular) data that come from other
  backend service(s) and are not currently accessible through PUG-View.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: PUG-View
  role: Web service used to issue the heading-filtered request and return the PUG-View
    JSON/XML content model
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:62ea876afb2da735a0677b91a03d0b6b7fde73409be8d907a9b5d9586abd7e2d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# json-annotation-data-parsing

## Summary

A method for retrieving and parsing heading-specific chemical annotations from PubChem records by issuing a PUG-View REST request to https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/ and validating the returned JSON against the PUG-View content model.

## When to use

Use when you need the annotation content (e.g. experimental properties, solubility, etc.) under a single specified (sub)heading for one PubChem record, returned as JSON or XML.

## When NOT to use

- A PUG-View request cannot take multiple input record identifiers — do not use this skill when you need to retrieve annotations for many records at once.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested — do not use this skill when you need several distinct headings for one record.
- Do not use this skill for specialized (especially tabular) data that come from other backend service(s) and are not currently accessible through PUG-View.

## Inputs

- PubChem record identifier (e.g. Compound ID 1983)
- Target (sub)heading string (e.g. 'Experimental Properties' or 'Solubility')
- Desired output format (JSON or XML)

## Outputs

- Valid JSON or XML file containing the annotation content restricted to the single requested (sub)heading for the specified record (e.g. solubility_cid1983.json)

## How to apply

Formulate a PUG-View request URL encoding three required pieces of information: the annotation type ('data'), the record of interest (e.g. Compound ID 1983), and the desired output format (JSON). Append the optional heading parameter (e.g. ?heading=Experimental+Properties) to the record URL, replacing any space characters with the '+' character per standard URL encoding; optionally verify the heading exists first via an 'index' request that returns the record's Table of Contents without full data. Issue an HTTP GET to https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON?heading=Solubility, noting the 30-second request time limit. Parse the returned PUG-View JSON/XML content model and save the heading-specific annotation data (e.g. as solubility_cid1983.json). Validate that the output is valid JSON whose annotation content is restricted to the single requested (sub)heading; if the compound has no data under that heading, the request returns an error message.

## Related tools

- **PUG-View** (Web service used to issue the heading-filtered request and return the PUG-View JSON/XML content model)

## Examples

```
curl 'https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON?heading=Solubility' -o solubility_cid1983.json
```

## Evaluation signals

- The output file must be valid JSON.
- The annotation content in the output must be restricted to the single requested (sub)heading.
- If the compound has no data in that section, the request returns an error message rather than empty data.
- The request URL correctly encodes the three required pieces of information: annotation type ('data'), the record of interest, and the desired output format.
- The request respects the 30-second time limit.

## Limitations

- The request has a 30-second time limit.
- If the compound has no data in the requested section, the request returns an error message.
- A PUG-View request cannot take multiple input record identifiers.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested.
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View.

## Evidence

- [other] Formulate a PUG-View request URL encoding the three required pieces of information: annotation type ('data'), the record of interest (Compound ID 1983), and the desired output format (JSON).: [evidence span withheld — anchor status: not_found]
- [other] A single-record PUG-View request accepts the optional heading parameter appended to the record URL ... with spaces URL-encoded as '+': [evidence span withheld — anchor status: not_found]
- [other] the request returns an error message if the compound has no data under that heading: [evidence span withheld — anchor status: not_found]
- [other] Validation: the output file must be valid JSON whose annotation content is restricted to the single requested (sub)heading: [evidence span withheld — anchor status: not_found]
- [discussion] The request time limit is 30 seconds.: [evidence span withheld — anchor status: not_found]
- [abstract] One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem.: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [discussion] a PUG-View request cannot take multiple input record identifiers: "a PUG-View request cannot take multiple input record identifiers"
