---
name: pubchem-record-identifier-handling
description: Use when a task requires retrieving PubChem annotations for one specific
  record via PUG-View and you must correctly encode the record identifier (e.g., CID),
  the output format, and an optional heading parameter into a request URL.
when_to_use_negative:
- When multiple record identifiers must be processed in one call — a PUG-View request
  cannot take multiple input record identifiers
- When annotations under multiple headings are needed in a single request, unless
  full annotation data retrieval is requested
- When accessing specialized (especially tabular) data that comes from other backend
  services and is not accessible through PUG-View
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: The REST-style web service interface whose URL syntax encodes the record identifier,
    heading, and output format to retrieve PubChem annotation data
- name: PUG-REST
  role: Complementary interface for accessing specific individual computed compound
    properties and for retrieving all neighbors (PUG-View neighbors form returns only
    neighbors with pre-defined annotation types)
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:75731e4f6ad32f448a395e65ca14f6ce634b886d9230dd649860698966ff6d5f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pubchem-record-identifier-handling

## Summary

Skill for constructing correct PUG-View request URLs that embed a single PubChem record identifier (e.g., a compound CID) and, optionally, a single URL-encoded (sub)heading, so annotation data can be retrieved programmatically as JSON or XML.

## When to use

Use when a task requires retrieving PubChem annotations for one specific record via PUG-View and you must correctly encode the record identifier (e.g., CID), the output format, and an optional heading parameter into a request URL.

## When NOT to use

- When multiple record identifiers must be processed in one call — a PUG-View request cannot take multiple input record identifiers
- When annotations under multiple headings are needed in a single request, unless full annotation data retrieval is requested
- When accessing specialized (especially tabular) data that comes from other backend services and is not accessible through PUG-View

## Inputs

- A single PubChem record identifier (e.g., a compound CID such as 1983 or 60823)
- A target (sub)heading string (e.g., Solubility, Experimental Properties, Biologic Description, Mass Spectrometry)
- A desired output format (JSON or XML)
- Optionally, an attachment key or internal biologic identifier obtained from a first-step PUG-View response

## Outputs

- A PUG-View JSON/XML content-model response restricted to the requested record and (sub)heading (e.g., solubility_cid1983.json)
- Error messages for records with no data under the requested heading
- A set of special-report responses keyed by ⟨ANNOTATION⟩: categories, neighbors, literature, structure, image, key, qr, linkout

## How to apply

Formulate a PUG-View request URL encoding three required pieces of information — the annotation type ('data'), the record of interest (e.g., Compound ID 1983), and the output format (JSON) — as in https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON. To restrict output to one section, append the optional heading parameter (e.g., ?heading=Solubility), replacing space characters with '+' per standard URL encoding; alternatively verify the heading exists first via an 'index' request returning the record's Table of Contents. Issue the HTTP GET within PUG-View's 30-second request time limit, then parse the returned PUG-View JSON/XML content model. For special-report variants, place the ⟨ANNOTATION⟩ value (categories, neighbors, literature, structure, image, key, qr, linkout) in the URL path; for two-step variants, first fetch the internal identifier (e.g., /data/compound/<CID>/JSON?heading=Biologic+Description returns the biologic identifier, e.g., 138308 for CID 91848714) or the attachment key (e.g., 3413726_1 from Mass Spectrometry), then issue the second call.

## Related tools

- **PUG-View** (The REST-style web service interface whose URL syntax encodes the record identifier, heading, and output format to retrieve PubChem annotation data)
- **PUG-REST** (Complementary interface for accessing specific individual computed compound properties and for retrieving all neighbors (PUG-View neighbors form returns only neighbors with pre-defined annotation types))

## Examples

```
curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON?heading=Solubility" -o solubility_cid1983.json
```

## Evaluation signals

- The request URL encodes the three required pieces of information: annotation type ('data'), the record identifier (e.g., compound/1983), and the output format (JSON)
- Space characters in the heading parameter are replaced with '+' (e.g., ?heading=Experimental+Properties)
- The returned file is valid JSON and its annotation content is restricted to the single requested (sub)heading
- A request for a heading under which the compound has no data returns an error message rather than empty content
- Second-step calls for two-step variants use the internal identifier from step one (e.g., /image/biologic/138308/SVG) or the attachment key with no output-format segment (mime type comes from the Content-Type header)

## Limitations

- A PUG-View request cannot take multiple input record identifiers
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested
- Requests are subject to a 30-second time limit
- Some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View
- The neighbors form returns only neighbors carrying pre-defined annotation types; PUG-REST is required for all neighbors
- NCBI Gene ID / protein accession identifiers are handled only via the data/ record form, not the special reports

## Evidence

- [other] heading parameter URL construction: [evidence span withheld — anchor status: not_found]
- [discussion] single-record constraint: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] single-heading constraint: "annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
- [intro] PUG-View scope: "which serves information needed to render interactive web pages but that also allows one to programmatically access the chemical annotations and summary information in PubChem"
- [other] two-step biologic depiction identifier: [evidence span withheld — anchor status: not_found]
