---
name: pug-view-url-construction
description: Use when a task requires programmatic retrieval of PubChem chemical annotations
  (e.g., solubility, LCSS, experimental properties) for one or a few record identifiers
  via PUG-View, and you need to formulate the correct REST request URL.
when_to_use_negative:
- When you need to batch-retrieve a property for many compound identifiers at once
  — PUG-View requests cannot take multiple input record identifiers per request; use
  PUG-REST instead for that.
- When the goal is to fetch computed compound properties (e.g., molecular weight,
  logP) — the article states PUG-REST is best for that; PUG-View is specialized for
  annotation content.
- When retrieving specialized tabular data not accessible through PUG-View — those
  come from other backend services.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3332
tools:
- name: PUG-View
  role: Primary tool used to construct and submit the REST-style request URL that
    encodes annotation type, record identifier, output format, and optional heading
    parameters.
- name: PUG-REST
  role: Alternative interface referenced for contrast; best for accessing specific
    individual computed compound properties and required for retrieving all neighbors
    rather than only those carrying pre-defined annotation types.
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/s13321-019-0375-2
    title: 'PUG-View: programmatic access to chemical annotations integrated in PubChem'
  source_content_hash: sha256:b07666943428b559035623a2aa83c370246ea6f02e332e822fc540d8bb4bf50c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PUG-REST
  - PUG-View
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pug-view-url-construction

## Summary

Constructs PUG-View request URLs by encoding the required annotation type, record identifier, and output format, plus optional parameters such as the heading or input string (X), to programmatically retrieve PubChem annotation data in JSON or XML.

## When to use

Use when a task requires programmatic retrieval of PubChem chemical annotations (e.g., solubility, LCSS, experimental properties) for one or a few record identifiers via PUG-View, and you need to formulate the correct REST request URL.

## When NOT to use

- When you need to batch-retrieve a property for many compound identifiers at once — PUG-View requests cannot take multiple input record identifiers per request; use PUG-REST instead for that.
- When the goal is to fetch computed compound properties (e.g., molecular weight, logP) — the article states PUG-REST is best for that; PUG-View is specialized for annotation content.
- When retrieving specialized tabular data not accessible through PUG-View — those come from other backend services.

## Inputs

- A PubChem record identifier (e.g., Compound CID 1983) or an X-form identifier triple (input, input_type, input_value)
- A target (sub)heading name for annotation retrieval, e.g. 'Solubility' or 'Experimental Properties'
- A desired output format: JSON or XML
- Optionally, an intermediate PUG-View JSON response containing an internal biologic identifier or annotation attachment key

## Outputs

- A PUG-View JSON or XML response containing the annotation data for the requested record and (sub)heading, e.g. solubility_cid1983.json
- An error message when the compound has no data under the requested heading
- A PUG-View 'index' response with the record's Table of Contents when the annotation type is 'index'
- A Content-Type-typed attachment (no JSON/XML format segment) when retrieving an annotation attachment via the data/key path

## How to apply

Build the request URL from its four parts: the base (https://pubchem.ncbi.nlm.nih.gov/rest/pug_view), the input (either an X-form 'input=compound&input_type=identifier&input_value=1983' or the /data/ record form 'data/compound/1983'), the annotation type ('data' for annotations vs. 'index' for the table of contents), and the output format (JSON or XML, appended as a URL path segment). Always append the optional heading parameter last, URL-encoding spaces as '+', e.g. 'heading=Experimental+Properties', to restrict the response to a single (sub)heading; the request returns an error if the compound has no data under that heading. For biologic depiction retrieval, first request the 'Biologic Description' heading to obtain the internal biologic identifier, then call the image path with that identifier; for annotation attachments, first request the relevant heading to get the attachment key, then call the data/key path with no output-format segment (the mime type comes from the Content-Type header). Use /data/ record form for record identifiers like CID; the special report variants (categories, neighbors, literature, structure, image, key, qr, linkout) are separate URL templates appended to the same base. Respect the 30-second request time limit and remember PUG-View cannot take multiple input record identifiers per request, so one request per record.

## Related tools

- **PUG-View** (Primary tool used to construct and submit the REST-style request URL that encodes annotation type, record identifier, output format, and optional heading parameters.)
- **PUG-REST** (Alternative interface referenced for contrast; best for accessing specific individual computed compound properties and required for retrieving all neighbors rather than only those carrying pre-defined annotation types.)

## Examples

```
curl -o solubility_cid1983.json 'https://pubchem.ncbi.nlm.nih.gov/rest/pug_view/data/compound/1983/JSON?heading=Solubility'
```

## Evaluation signals

- The returned file is valid JSON (or XML) and its annotation content is restricted to the single requested (sub)heading — e.g., solubility_cid1983.json contains only solubility annotations for CID 1983.
- The request URL contains exactly one record identifier; PUG-View does not accept multiple input record identifiers in a single request.
- Spaces in the heading parameter are encoded as '+' (standard URL encoding), e.g., 'heading=Experimental+Properties'.
- For attachment/image retrieval, the intermediate request successfully returned an internal identifier or attachment key (e.g., biologic id 138308 for CID 91848714, or key 3413726_1) before the second call was made.
- Attachment data retrieved via the data/key path is delivered with the correct Content-Type (no output-format segment in the URL) rather than as JSON/XML.

## Limitations

- PUG-View requests have a 30-second time limit.
- A PUG-View request cannot take multiple input record identifiers.
- Annotation data under multiple headings for a given compound cannot be retrieved with a single request unless the full annotation data retrieval is requested.
- Some specialized (especially tabular) data are not currently accessible through PUG-View.
- If the compound has no data under the requested (sub)heading, the request returns an error message.

## Evidence

- [methods] A single-record PUG-View request accepts the optional heading parameter appended to the record URL (e.g., /data/compound/1983/JSON?heading=Experimental+Properties, with spaces URL-encoded as '+') to return only the annotations under that (sub)heading: [evidence span withheld — anchor status: not_found]
- [methods] if the compound has no data in that section, the request returns an error message: [evidence span withheld — anchor status: not_found]
- [methods] noting the 30-second request time limit: [evidence span withheld — anchor status: not_found]
- [discussion] a PUG-View request cannot take multiple input record identifiers: "a PUG-View request cannot take multiple input record identifiers"
- [discussion] Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested: "Annotation data under multiple headings for a given compound cannot be retrieved with a single request, unless the full annotation data retrieval is requested"
- [discussion] some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View: "some specialized (especially tabular) data come from other backend service(s) and are not currently accessible through PUG-View"
- [abstract] One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem.: "One of these is PUG-View, which is a Representational State Transfer (REST)-style web service interface specialized for accessing annotation data contained in PubChem."
