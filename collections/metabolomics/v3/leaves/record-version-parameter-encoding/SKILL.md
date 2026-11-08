---
name: record-version-parameter-encoding
description: Reference for retrieving historical versions of PubChem substance (SID)
  or assay (AID) records when the current annotation is insufficient and an earlier
  depositor-provided record version is needed.
when_to_use_negative:
- When the goal is to retrieve specific individual computed compound properties —
  PUG-REST is better suited, while PUG-View is specialized for annotation data
- When multiple record identifiers need to be fetched in one request — a PUG-View
  request cannot take multiple input record identifiers, so one request per record/version
  is required
- When the target is specialized tabular data, which comes from other backend services
  and is not currently accessible through PUG-View
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: PUG-View
  role: REST-style web service whose /rest/pug_view/data/{substance|assay}/{ID}/JSON
    endpoint accepts the optional version query parameter used to retrieve historical
    record annotations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:e6935b86b3fb1285f41db8401c143279c35747374b0ece664d4557ed840563df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-View
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# record-version-parameter-encoding

## Summary

Retrieves previous versions of depositor-provided PubChem substance or assay annotation records through PUG-View by appending an optional version parameter to the request URL. This skill captures the version-number syntax difference between substances (integer form) and assays (n.m form).

## When to use

Reference for retrieving historical versions of PubChem substance (SID) or assay (AID) records when the current annotation is insufficient and an earlier depositor-provided record version is needed.

## When NOT to use

- When the goal is to retrieve specific individual computed compound properties — PUG-REST is better suited, while PUG-View is specialized for annotation data
- When multiple record identifiers need to be fetched in one request — a PUG-View request cannot take multiple input record identifiers, so one request per record/version is required
- When the target is specialized tabular data, which comes from other backend services and is not currently accessible through PUG-View

## Inputs

- PubChem substance identifier (SID), e.g. 46507068, with a target integer version number
- PubChem assay identifier (AID), e.g. 369, with a target version number in n.m form (e.g. 1.1)
- PUG-View request URL prefix: https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/{record_type}/{ID}/JSON

## Outputs

- JSON annotation files containing the requested historical version of the substance record (e.g. substance_46507068_version_2.json)
- JSON annotation files containing the requested historical version of the assay record (e.g. assay_369_version_1.1.json)

## How to apply

Construct a PUG-View REST request URL using the standard annotation-data endpoint pattern /rest/pug_view/data/substance/{SID}/JSON or /rest/pug_view/data/assay/{AID}/JSON, then append the optional version parameter as a query string: ?version=2 for a substance record (integer version number) and ?version=1.1 for an assay record, since assay version numbers use the n.m form. Issue HTTP GET requests to the constructed URLs, throttled to at most 5 requests per second per the PubChem usage policy, and save each returned annotation payload to a named JSON file (e.g., substance_46507068_version_2.json, assay_369_version_1.1.json). Validate that each response parses as valid JSON and represents the requested record version rather than a PubChem error message.

## Related tools

- **PUG-View** (REST-style web service whose /rest/pug_view/data/{substance|assay}/{ID}/JSON endpoint accepts the optional version query parameter used to retrieve historical record annotations)

## Examples

```
curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/substance/46507068/JSON?version=2" -o substance_46507068_version_2.json; curl "https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/assay/369/JSON?version=1.1" -o assay_369_version_1.1.json
```

## Evaluation signals

- Each saved response parses as valid JSON (e.g. loads with json.load) without a parse error
- The parsed response represents the requested record version rather than a PubChem error message
- Substance requests use integer version parameters (?version=2) while assay requests use n.m-form version parameters (?version=1.1); mismatches indicate malformed URLs
- Request rate does not exceed 5 requests per second, per PubChem usage policy

## Limitations

- A PUG-View request cannot take multiple input record identifiers, so each versioned record must be requested individually
- Some specialized (especially tabular) annotation data are served by other backend services and are not currently accessible through PUG-View
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request unless full annotation data retrieval is requested

## Evidence

- [other] PUG-View retrieves previous versions of substance or assay records by passing the version number as an optional parameter in the request URL: [evidence span withheld — anchor status: not_found]
- [other] version syntax differs between substances and assays: [evidence span withheld — anchor status: not_found]
- [abstract] PUG-View is a REST-style interface specialized for annotation data: "PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
- [intro] PUG-View request syntax is a documented aspect of the interface: "The present paper will describe important aspects of PUG-View, including the scope of data accessible through PUG-View, the syntax for formulation a PUG-View request URL"
